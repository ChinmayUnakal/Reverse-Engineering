# Program 02 — Integer Assignment

## 1. Objective

Understand how C integer initialization and assignment translate into Windows x64 assembly.

Skills:

- Recognize immediate integer values in hexadecimal.
    
- Understand `mov` between memory and registers.
    
- Identify local variables stored relative to `RBP`.
    
- Trace an assignment from a source variable to a destination variable.
    
- Recognize function prologue, epilogue, and compiler-generated runtime initialization.
    

## 2. Original C

```
#include <stdio.h>

int main(void)
{
    int a = 10;
    int b = 20;
    int c = a;

    return 0;
}
```

## 3. Behavioral prediction

- **Expected output:** No output.
    
- **Expected return value:** `0`.
    
- **Important variable values:**
    
    - `a = 10`
        
    - `b = 20`
        
    - `c = 10`
        
- **Expected branches or function calls:** No explicit branches or library calls in the C source. The generated binary calls GCC's `__main` runtime helper.
    
- **Expected memory:** The generated assembly stores the local integer values at stack-relative addresses.
    

## 4. Compilation

- **Compiler:** GCC gcc.exe (Rev13, Built by MSYS2 project) 15.2.0
    
- **Architecture:** Windows x64.
    
- **Build configuration:** `-O0 -g`.
    
- **Compilation command:**
    

```
gcc -O0 -g -o int_assign.exe int_assign.c
```

- **Debugger:** WinDbg.
    
- **Observed function:** `int_assign!main`.
    

## 5. Assembly observations

### A. Function prologue

```
00007ff6`fb861490  55             push rbp
00007ff6`fb861491  4889e5         mov  rbp, rsp
00007ff6`fb861494  4883ec30       sub  rsp, 30h
```

|Instruction|Register or memory effect|My interpretation|
|---|---|---|
|`push rbp`|Saves the previous `RBP` on the stack.|Preserves the caller's frame pointer.|
|`mov rbp, rsp`|Copies `RSP` into `RBP`.|Establishes a reference point for this function's stack frame.|
|`sub rsp, 30h`|Decreases `RSP` by `0x30` = 48 bytes.|Reserves stack space for the function's needs and calling-convention requirements.|

**Observation:** The function reserves 48 bytes, even though the three declared integers require only 12 bytes of storage. The entire reserved area is not necessarily dedicated to those three variables; alignment and call-related requirements also affect the stack frame.

### B. GCC runtime initialization

```
00007ff6`fb861498  e8f3000000  call int_assign!__main
```

- `call` transfers execution to the `__main` helper.
    
- The helper is part of the GCC runtime startup support; it is not the C function named `main`.
    
- After the helper returns, execution continues at `0x00007ff6`fb86149d`.
    

**RE lesson:** Compiler-generated assembly can contain runtime setup that does not correspond directly to a statement in the C source.

### C. Initialize `a` to 10

```
00007ff6`fb86149d  c745fc0a000000  mov dword ptr [rbp-4], 0Ah
```

Breakdown:

- `mov` copies a value into the destination.
    
- `dword ptr` specifies a 4-byte memory operand.
    
- `[rbp-4]` refers to the memory address four bytes below the address in `RBP`.
    
- `0Ah` is hexadecimal for decimal `10`.
    

**Interpretation:** The instruction stores the 32-bit value `10` at `[RBP-4]`. Based on the source and instruction sequence, this is the stack slot used for `a`.

### D. Initialize `b` to 20

```
00007ff6`fb8614a4  c745f814000000  mov dword ptr [rbp-8], 14h
```

Breakdown:

- `[rbp-8]` is a stack-relative memory location eight bytes below `RBP`.
    
- `14h` is hexadecimal for decimal `20`.
    
- The instruction stores a 32-bit value directly into memory.
    

**Interpretation:** The compiler initializes `b` to `20` at `[RBP-8]`.

### E. Read `a` into `EAX`

```
00007ff6`fb8614ab  8b45fc  mov eax, dword ptr [rbp-4]
```

- The source operand is the 4-byte value at `[RBP-4]`.
    
- The destination is `EAX`, the lower 32 bits of `RAX`.
    
- Because the previous instruction stored `10` there, `EAX` receives `10`.
    

**Interpretation:** The program reads the current value of `a` into a register.

Conceptually:

```
eax = a;
```

This is explanatory pseudocode, not literal C syntax for a CPU register.

### F. Assign the value to `c`

```
00007ff6`fb8614ae  8945f4  mov dword ptr [rbp-0Ch], eax
```

- `[RBP-0Ch]` means a stack-relative location 12 bytes below `RBP`; `0Ch` is hexadecimal for decimal `12`.
    
- `EAX` contains `10` from the previous instruction.
    
- The instruction writes that value into the destination memory slot.
    

**Interpretation:** The compiler stores the value of `a` into the stack slot used for `c`.

The combined data flow is:

```
[RBP-4] = 10        ; a = 10
[RBP-8] = 20        ; b = 20
EAX = [RBP-4]       ; read a into EAX
[RBP-0Ch] = EAX     ; c = a
```

After these instructions:

|Variable|Stack location|Value|
|---|---|---|
|`a`|`[RBP-4]`|10|
|`b`|`[RBP-8]`|20|
|`c`|`[RBP-0Ch]`|10|

These variable-to-slot associations are inferred from the observed instructions and source. Stack offsets are not universal identifiers for C variables.

### G. Set the return value

```
00007ff6`fb8614b1  b800000000  mov eax, 0
```

- `EAX` is set to zero.
    
- On Windows x64, integer return values are returned in `EAX`/`RAX`, depending on their type and width.
    

**Interpretation:** This sets the return value of `main` to `0`.

Notice that `EAX` previously held `10`, but its value is overwritten here. The program no longer needs the value of `a` for its return value.

### H. Function epilogue

```
00007ff6`fb8614b6  4883c430  add rsp, 30h
00007ff6`fb8614ba  5d        pop rbp
00007ff6`fb8614bb  c3        ret
```

|Instruction|Effect|My interpretation|
|---|---|---|
|`add rsp, 30h`|Adds 48 to `RSP`.|Releases the reserved stack space.|
|`pop rbp`|Restores the previous `RBP`.|Restores the caller's frame pointer.|
|`ret`|Transfers control to the saved return address.|Returns to the caller.|

## 6. C reconstruction

Reconstructed from the observed data flow:

```
int main(void)
{
    int a = 10;
    int b = 20;
    int c = 10;

    return 0;
}
```

This reconstruction is consistent with the assembly. The assignments to `a`, `b`, and `c` are visible in the generated instructions.

However, these local variables are never used in any observable way, so a different compiler configuration could eliminate the assignments. The machine code does not guarantee that every source-level variable survives compilation.

## 7. Verification

- The binary was compiled and disassembled in WinDbg.
    
- The assembly contains a store of `10` at `[RBP-4]`.
    
- The assembly contains a store of `20` at `[RBP-8]`.
    
- The assembly loads `a` into `EAX`.
    
- The assembly stores the value from `EAX` into `[RBP-0Ch]`.
    
- The assembly sets the return value to `0`.
    
- Compile the reconstructed source and execute it.
    
- Compare the reconstructed binary with the original.
    
- Compile with `-O2` and observe whether the assignments remain.
    

**Expected runtime behavior:** No output and an exit status of `0`.

## 8. What I misunderstood

- I did not initially understand why `c = a` involves a load into a register followed by a store to memory. This happened because inside the program I had c = a; so instead of directly doing 
  c = 10; c was assigned variable a.

## 9. Key RE lesson

Integer initialization can appear as an immediate value written directly into stack memory. A normal integer assignment can involve loading the source value into a register and storing it at the destination address. In this binary, `a` is stored at `[RBP-4]`, read into `EAX`, and copied into the stack slot for `c`. The stack-frame size is larger than the total storage used by these three integers because the frame also accommodates other function requirements.

## 10. Checkpoint

- I can explain `mov dword ptr [rbp-4], 0Ah`.
    
- I can convert `0Ah`, `14h`, and `0Ch` from hexadecimal to decimal.
    
- I can explain why `c = a` becomes a load followed by a store.
    
- I can track the values of `a`, `b`, and `c` through the instructions.
    
- I can explain the prologue, runtime helper call, and epilogue.
    
- I can reconstruct the program without consulting the original source.
    
- I can explain why an optimized build might remove the assignments.
    
- I can solve a slightly modified integer-assignment program independently.