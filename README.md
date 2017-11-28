# control flow flattening

## idea

Obfuscate control flow by introducing a loop and state machine (moore) which dispatches the basic blocks. This dispatcher is _very_ often implemented as a switch in high-level code. 

Why do it?

## obfuscation

Security through obscurity?

Buying time?

Track record of obfuscation techniques?

## algorithm
A flattening transform implementation could, in the spirit of go, look something like this:
```go
/* a simple flattening algorithm */
func Flatten(f: ir.Function)
    blocks       := f.BasicBlocks()
    switch_var   := ir.Var(types.Int32)
    switch_instr := ir.Switch(switch_var)
    loop_instr   := ir.While(ir.Bool.True)
    
    loop_instr.Add(switch_instr)
    
    /* map each top level block to its own case label */
    for i, block := range(blocks) {  
        /* later on we want to sprinkle some magic on this computation */
        next_block_id   := i + 1
        next_block_expr := switch_var.Store(next_block_id)

        block.Add(next_block_expr)
        block.Add(switch_instr.Break())
        switch_instr.Case(i, block)
    }
    
    f.BasicBlocks().Empty()
    f.Add(switch_var.Decl(0))
    f.Add(loop_instr)
}
```

### example (code)
Here is a very simple example (with some sketchy x86 assembly).
```go
/* original function, { ... } to force flattening  */
func Magic(x: int) int
{
                          // mov eax, x
    { x = x + 1; }        // add eax, 1
    { x = x % 3; }        // mod eax, 3
    { x = x * 2; }        // mul eax, 2
                          // mov ??, eax
    return x;             // ret
}

/* transforms into this monstrosity */
func Magic(x: int) int
{                       
    block := 0;           // mov block, 0
    while (true) {        // label loop_0_start
        switch (block) {  // label loop_0_switch_0_start
                          //     # jump to case impl.
        case 0:           // label loop_0_switch_0_case_0
            x = x + 1;    //     mov eax, x
                          //     add eax, 1
            block = 1;    //     mov block, 1
            break;        //     jmp loop_0_switch_0_end
        case 1:           // label loop_0_switch_0_case_1
            x = x % 3;    //     mod eax, 3
            block = 2;    //     mov block, 3
            break;        //     jmp loop_0_switch_0_end
        case 2:           // label loop_0_switch_0_case_2
            x = x * 2;    //     mul eax, 2
            block = 3;    //     mov block, 3
            break;        //     jmp loop_0_switch_0_end
        case 3:           // label loop_0_switch_0_case_3
                          //     mov ??, eax
            return x;     //     ret
        }
                          // label loop_0_switch_0_end
                          //     jmp loop_0_start
    }
}
```

### example (graph)
The control flow graphs:

![img 1]()
![img 2]()

## attacks

 - symbolic execution + analysis of switch var
 - code lifting
 - piggyback on the compiler:  loop `asm to ir -> recompile w/optimization -> asm to ir`

## improving

 - scramble case order
 - split (large) basic blocks
 - duplicate blocks
 - dummy (bogus) blocks
 - opaque predicates
 - dispatcher obfuscation

## tools

## results

## summary


## links

 - [Obfuscating C++ programs via control flow flattening, T László and A Kiss, 2009](http://ac.inf.elte.hu/Vol_030_2009/003.pdf)
