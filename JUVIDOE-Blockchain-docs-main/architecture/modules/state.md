# State

To truly understand how **State** works, you must understand some basic Juvidoe concepts.\\

We highly recommend reading the **State in Juvidoe guide**.

### Overview

Now that we've familiarized ourselves with basic Juvidoe concepts, the next overview should be easy.

We mentioned that the **World state trie** has all the Juvidoe accounts that exist.\
These accounts are the leaves of the Merkle trie. Each leaf has encoded **Account State** information.

This enables the Juvidoe Edge to get a specific Merkle trie, for a specific point in time.\
For example, we can get the hash of the state at block 10.

The Merkle trie, at any point in time, is called a _**Snapshot**_.

We can have _**Snapshots**_ for the **state trie**, or for the **storage trie** - they are basically the same.\
The only difference is in what the leaves represent:

* In the case of the storage trie, the leaves contain an arbitrary state, which we cannot process or know what's in there
* In the case of the state trie, the leaves represent accounts

"state/state.go

```
type State interface {
    // Gets a snapshot for a specific hash
    NewSnapshotAt(types.Hash) (Snapshot, error)
    
    // Gets the latest snapshot
    NewSnapshot() Snapshot
    
    // Gets the codeHash
    GetCode(hash types.Hash) ([]byte, bool)
}
```

The **Snapshot** interface is defined as such:

"state/state.go

```
type Snapshot interface {
    // Gets a specific value for a leaf
    Get(k []byte) ([]byte, bool)
    
    // Commits new information
    Commit(objs []*Object) (Snapshot, []byte)
}
```

The information that can be committed is defined by the _Object struct_:

"state/state.go

```
// Object is the serialization of the radix object
type Object struct {
    Address  types.Address
    CodeHash types.Hash
    Balance  *big.Int
    Root     types.Hash
    Nonce    uint64
    Deleted  bool

    DirtyCode bool
    Code      []byte

    Storage []*StorageObject
}
```

The implementation for the Merkle trie is in the _state/immutable-trie_ folder.\
\&#xNAN;_state/immutable-trie/state.go_ implements the **State** interface.

_state/immutable-trie/trie.go_ is the main Merkle trie object. It represents an optimized version of the Merkle trie, which reuses as much memory as possible.

### Executor

_state/executor.go_ includes all the information needed for the Juvidoe Edge to decide how a block changes the current state. The implementation of _ProcessBlock_ is located here.

The _apply_ method does the actual state transition. The executor calls the JVM.

state/executor.go

```
func (t *Transition) apply(msg *types.Transaction) ([]byte, uint64, bool, error) {
    // check if there is enough gas in the pool
    if err := t.subGasPool(msg.Gas); err != nil {
        return nil, 0, false, err
    }

    txn := t.state
    s := txn.Snapshot()

    gas, err := t.preCheck(msg)
    if err != nil {
        return nil, 0, false, err
    }
    if gas > msg.Gas {
        return nil, 0, false, errorVMOutOfGas
    }

    gasPrice := new(big.Int).SetBytes(msg.GetGasPrice())
    value := new(big.Int).SetBytes(msg.Value)

    // Set the specific transaction fields in the context
    t.ctx.GasPrice = types.BytesToHash(msg.GetGasPrice())
    t.ctx.Origin = msg.From

    var subErr error
    var gasLeft uint64
    var returnValue []byte

    if msg.IsContractCreation() {
        _, gasLeft, subErr = t.Create2(msg.From, msg.Input, value, gas)
    } else {
        txn.IncrNonce(msg.From)
        returnValue, gasLeft, subErr = t.Call2(msg.From, *msg.To, msg.Input, value, gas)
    }
    
    if subErr != nil {
        if subErr == runtime.ErrNotEnoughFunds {
            txn.RevertToSnapshot(s)
            return nil, 0, false, subErr
        }
    }

    gasUsed := msg.Gas - gasLeft
    refund := gasUsed / 2
    if refund > txn.GetRefund() {
        refund = txn.GetRefund()
    }

    gasLeft += refund
    gasUsed -= refund

    // refund the sender
    remaining := new(big.Int).Mul(new(big.Int).SetUint64(gasLeft), gasPrice)
    txn.AddBalance(msg.From, remaining)

    // pay the coinbase
    coinbaseFee := new(big.Int).Mul(new(big.Int).SetUint64(gasUsed), gasPrice)
    txn.AddBalance(t.ctx.Coinbase, coinbaseFee)

    // return gas to the pool
    t.addGasPool(gasLeft)

    return returnValue, gasUsed, subErr != nil, nil
}
```

### Runtime

When a state transition is executed, the main module that executes the state transition is the jVM (located in state/runtime/jvm).

The **dispatch table** does a match between the **opcode** and the instruction.

state/runtime/jvm/dispatch\_table.go

```
func init() {
    // unsigned arithmetic operations
    register(STOP, handler{opStop, 0, 0})
    register(ADD, handler{opAdd, 2, 3})
    register(SUB, handler{opSub, 2, 3})
    register(MUL, handler{opMul, 2, 5})
    register(DIV, handler{opDiv, 2, 5})
    register(SDIV, handler{opSDiv, 2, 5})
    register(MOD, handler{opMod, 2, 5})
    register(SMOD, handler{opSMod, 2, 5})
    register(EXP, handler{opExp, 2, 10})

    ...

    // jumps
    register(JUMP, handler{opJump, 1, 8})
    register(JUMPI, handler{opJumpi, 2, 10})
    register(JUMPDEST, handler{opJumpDest, 0, 1})
}
```

The core logic that powers the JVM is the _Run_ loop.\\

This is the main entry point for the JVM. It does a loop and checks the current opcode, fetches the instruction, checks if it can be executed, consumes gas and executes the instruction until it either fails or stops.

state/runtime/jvm/state.go

```

// Run executes the virtual machine
func (c *state) Run() ([]byte, error) {
    var vmerr error

    codeSize := len(c.code)
    
    for !c.stop {
        if c.ip >= codeSize {
            c.halt()
            break
        }

        op := OpCode(c.code[c.ip])

        inst := dispatchTable[op]
        
        if inst.inst == nil {
            c.exit(errOpCodeNotFound)
            break
        }
        
        // check if the depth of the stack is enough for the instruction
        if c.sp < inst.stack {
            c.exit(errStackUnderflow)
            break
        }
        
        // consume the gas of the instruction
        if !c.consumeGas(inst.gas) {
            c.exit(errOutOfGas)
            break
        }

        // execute the instruction
        inst.inst(c)

        // check if stack size exceeds the max size
        if c.sp > stackSize {
            c.exit(errStackOverflow)
            break
        }
        
        c.ip++
    }

    if err := c.err; err != nil {
        vmerr = err
    }
    
    return c.ret, vmerr
}
```
