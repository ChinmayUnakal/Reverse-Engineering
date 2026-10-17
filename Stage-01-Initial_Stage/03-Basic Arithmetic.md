# Program 03 — Basic Arithmetic

## 1. Objective

This program teaches how to reverse-engineer basic arithmetic operations from x86-64 assembly and reconstruct equivalent C source code.

By completing this program, I should be able to:

- Identify local variables stored in stack memory using offsets from `RBP`.
    
- Recognize addition (`ADD`), subtraction (`SUB`), and multiplication (`IMUL`).
    
- Track how values move between registers such as `EAX` and `EDX`.
    
- Reconstruct arithmetic expressions from assembly instructions.
    
- Understand how the Windows x64 calling convention passes arguments to `printf`.
    
- Distinguish between a program's printed output and its return value.
    

## 2. Original C

```
#include <stdio.h>

int main(void)
{
    int a = 20;
    int b = 5;

    int x = a + b;
    int y = a - b;
    int z = a * b;

    int result = x + y + z;

    printf("Result = %d\n", result);

    return 0;
}
```

## 3. Behavioral Prediction

### Expected output

```
Result = 140
```

### Expected return value

`0`, because `main` executes `return 0;`.

The arithmetic result is printed to the console; it is not the program's exit status.

### Important variable values

|Variable|Expression|Expected value|
|---|---|---|
|`a`|Initialized|20|
|`b`|Initialized|5|
|`x`|`a + b`|25|
|`y`|`a - b`|15|
|`z`|`a * b`|100|
|`result`|`x + y + z`|140|

### Expected branches and function calls

- No conditional branches are expected.
    
- `__main` is called as a GCC runtime initialization helper.
    
- `printf` is called to display the arithmetic result.
    
- `main` returns `0` after printing.
    

## 4. Compilation

- **Compiler:** GCC gcc.exe (Rev13, Built by MSYS2 project) 15.2.0
    
- **Compiler version:** Not recorded
    
- **Architecture:** x86-64
    
- **Operating system:** Windows
    
- **Build configuration:** `-O0 -g`
    
- **Optimization:** Disabled (`-O0`)
    
- **Debug information:** Enabled (`-g`)
    

### Compilation command

```
gcc -O0 -g -o basic_arith.exe 03_basic_arithmetic.c
```

Expected output:

```
Result = 140
```

Check the process exit status in PowerShell:

```
$LASTEXITCODE
```

Expected exit status:

```
0
```

### Debugging environment

- WinDbg
    
- Windows x64 target
    
- Disassembly of `main`
    

## 5. Assembly Observations

### 5.1 Function prologue and stack allocation

```
push rbp
mov rbp, rsp
sub rsp, 40h
call basic_arith!__main
```

|Instruction|Register or memory effect|Interpretation|
|---|---|---|
|`push rbp`|Saves the previous `RBP` on the stack|Preserves the caller's frame pointer.|
|`mov rbp, rsp`|Copies `RSP` into `RBP`|Establishes a reference point for local variables.|
|`sub rsp, 40h`|Reserves `0x40` = 64 bytes|Allocates stack space for local variables, call requirements, and alignment.|
|`call basic_arith!__main`|Calls a GCC helper|Runtime initialization; not an arithmetic operation.|

**Important:** In this function, `RBP` acts as the frame pointer. Memory references such as `[rbp-4]` access locations below the frame pointer.

### 5.2 Mapping stack variables

The following stack locations were identified in the supplied disassembly:

|Stack location|Reconstructed variable|Value|
|---|---|---|
|`[rbp-4]`|`a`|20 (`14h`)|
|`[rbp-8]`|`b`|5|
|`[rbp-0Ch]`|`x`|25|
|`[rbp-10h]`|`y`|15|
|`[rbp-14h]`|`z`|100|
|`[rbp-18h]`|`result`|140|

The mappings are inferred by tracking how values are stored and subsequently used.

The original variable names are known here because the source code is available for comparison. Without source or debugging information, the exact names generally cannot be recovered from assembly alone.

### 5.3 Initializing `a` and `b`

```
mov dword ptr [rbp-4], 14h
mov dword ptr [rbp-8], 5
```

|Instruction|Meaning|
|---|---|
|`mov dword ptr [rbp-4], 14h`|Stores decimal `20` in the stack slot for `a`.|
|`mov dword ptr [rbp-8], 5`|Stores decimal `5` in the stack slot for `b`.|

`14h` is hexadecimal for decimal `20`.

`dword ptr` specifies a 32-bit memory operand, matching the size of an ordinary 32-bit `int` on this target.

Equivalent C:

```
int a = 20;
int b = 5;
```

### 5.4 Addition — `x = a + b`

```
mov edx, dword ptr [rbp-4]
mov eax, dword ptr [rbp-8]
add eax, edx
mov dword ptr [rbp-0Ch], eax
```

#### Register trace

|Step|Instruction|`EAX`|`EDX`|
|---|---|---|---|
|1|`mov edx, [rbp-4]`|—|20|
|2|`mov eax, [rbp-8]`|5|20|
|3|`add eax, edx`|25|20|
|4|`mov [rbp-0Ch], eax`|25|20|

The key instruction is:

```
add eax, edx
```

Its operation is:

```
EAX = EAX + EDX
EAX = 5 + 20
EAX = 25
```

The result is stored at `[rbp-0Ch]`, corresponding to `x`.

Equivalent C:

```
int x = a + b;
```

**Observation:** The compiler loads `a` into `EDX` and `b` into `EAX`. This is valid because addition is commutative: `5 + 20` equals `20 + 5`.

### 5.5 Subtraction — `y = a - b`

```
mov eax, dword ptr [rbp-4]
sub eax, dword ptr [rbp-8]
mov dword ptr [rbp-10h], eax
```

#### Register trace

|Step|Instruction|`EAX`|
|---|---|---|
|1|`mov eax, [rbp-4]`|20|
|2|`sub eax, [rbp-8]`|15|
|3|`mov [rbp-10h], eax`|15|

The key instruction is:

```
sub eax, dword ptr [rbp-8]
```

Its operation is:

```
EAX = EAX - [rbp-8]
EAX = 20 - 5
EAX = 15
```

The result is stored at `[rbp-10h]`, corresponding to `y`.

Equivalent C:

```
int y = a - b;
```

**Observation:** Subtraction is not commutative. Operand order matters: `20 - 5` is not equivalent to `5 - 20`.

### 5.6 Multiplication — `z = a * b`

```
mov eax, dword ptr [rbp-4]
imul eax, dword ptr [rbp-8]
mov dword ptr [rbp-14h], eax
```

#### Register trace

|Step|Instruction|`EAX`|
|---|---|---|
|1|`mov eax, [rbp-4]`|20|
|2|`imul eax, [rbp-8]`|100|
|3|`mov [rbp-14h], eax`|100|

The important instruction is:

```
imul eax, dword ptr [rbp-8]
```

In this two-operand form, the operation is:

```
EAX = EAX * [rbp-8]
EAX = 20 * 5
EAX = 100
```

The result is stored at `[rbp-14h]`, corresponding to `z`.

Equivalent C:

```
int z = a * b;
```

**Observation:** `IMUL` performs signed integer multiplication. The two-operand form writes the low 32 bits of the product to `EAX`.

A compiler does not always need an explicit multiplication instruction. Depending on the expression and optimization settings, multiplication may also be implemented using `LEA`, shifts, or additions.

### 5.7 Calculating `result = x + y + z`

```
mov edx, dword ptr [rbp-0Ch]
mov eax, dword ptr [rbp-10h]
add edx, eax
mov eax, dword ptr [rbp-14h]
add eax, edx
mov dword ptr [rbp-18h], eax
```

#### Register trace

|Step|Operation|`EAX`|`EDX`|
|---|---|---|---|
|1|Load `x` into `EDX`|—|25|
|2|Load `y` into `EAX`|15|25|
|3|`add edx, eax`|15|40|
|4|Load `z` into `EAX`|100|40|
|5|`add eax, edx`|140|40|
|6|Store `EAX` in `result`|140|40|

First addition:

```
EDX = 25 + 15
EDX = 40
```

Second addition:

```
EAX = 100 + 40
EAX = 140
```

The value `140` is stored at `[rbp-18h]`, corresponding to `result`.

Equivalent C:

```
int result = x + y + z;
```

**Observation:** Do not assume that the assembly will follow the exact visual order of the C expression. Track the values through registers and memory to reconstruct the data flow.

### 5.8 Preparing the arguments for `printf`

```
mov eax, dword ptr [rbp-18h]
lea rcx, [7FF6882A4000h]
mov edx, eax
call basic_arith!printf
```

This introduces the Windows x64 calling convention.

For ordinary integer and pointer arguments, the first four argument positions use these registers:

|Argument position|Register|
|---|---|
|First|`RCX`|
|Second|`RDX`|
|Third|`R8`|
|Fourth|`R9`|

#### Instruction-by-instruction interpretation

|Instruction|Meaning|
|---|---|
|`mov eax, [rbp-18h]`|Loads `result`, which is `140`, into `EAX`.|
|`lea rcx, [7FF6882A4000h]`|Loads the address of the format string into `RCX`.|
|`mov edx, eax`|Copies the integer result, `140`, into `EDX`.|
|`call basic_arith!printf`|Calls `printf` with the prepared arguments.|

#### Argument mapping

|Register|Argument|Expected value|
|---|---|---|
|`RCX`|First argument: format string|Address of `"Result = %d\n"`|
|`RDX`|Second argument: value for `%d`|`140`|

The reconstructed function call is:

```
printf("Result = %d\n", result);
```

**Important distinction:** `LEA` calculates an address; it does not read the string contents into `RCX`.

To verify the exact string in WinDbg, inspect the address from the disassembly:

```
da 00007ff6882a4000
```

Use the actual address from the current debugging session if it differs.

### 5.9 Returning from `main`

```
mov eax, 0
add rsp, 40h
pop rbp
ret
```

|Instruction|Meaning|
|---|---|
|`mov eax, 0`|Sets the return value of `main` to zero.|
|`add rsp, 40h`|Releases the 64 bytes of stack space reserved earlier.|
|`pop rbp`|Restores the previous frame pointer.|
|`ret`|Returns to the caller.|

On x64 Windows, an integer return value is delivered in `EAX`.

The program is **not** returning `140`. That value was passed to `printf`. Before returning, the program explicitly sets `EAX` to `0`.

Therefore:

- Printed result: `140`
    
- Return value: `0`
    
- Process exit status: `0`
    

### 5.10 Complete assembly interpretation table

| Instruction/address                      | Register or memory effect        | Interpretation                                  |
| ---------------------------------------- | -------------------------------- | ----------------------------------------------- |
| `00007ff6\`882a1490`—`push rbp`          | Saves previous `RBP`             | Function prologue                               |
| `00007ff6\`882a1491`—`mov rbp,rsp`       | Establishes frame pointer        | Stack frame setup                               |
| `00007ff6\`882a1494`—`sub rsp,40h`       | Reserves 64 bytes                | Local storage and call requirements             |
| `00007ff6\`882a1498`—`call __main`       | Calls GCC helper                 | Runtime initialization                          |
| `00007ff6\`882a149d`—`mov [rbp-4],14h`   | Stores 20                        | Initializes `a`                                 |
| `00007ff6\`882a14a4`—`mov [rbp-8],5`     | Stores 5                         | Initializes `b`                                 |
| `00007ff6\`882a14ab`—`mov edx,[rbp-4]`   | `EDX = 20`                       | Loads `a`                                       |
| `00007ff6\`882a14ae`—`mov eax,[rbp-8]`   | `EAX = 5`                        | Loads `b`                                       |
| `00007ff6\`882a14b1`—`add eax,edx`       | `EAX = 25`                       | Calculates `x`                                  |
| `00007ff6\`882a14b3`—`mov [rbp-0Ch],eax` | Stores 25                        | Saves `x`                                       |
| `00007ff6\`882a14b6`—`mov eax,[rbp-4]`   | `EAX = 20`                       | Loads `a`                                       |
| `00007ff6\`882a14b9`—`sub eax,[rbp-8]`   | `EAX = 15`                       | Calculates `y`                                  |
| `00007ff6\`882a14bc`—`mov [rbp-10h],eax` | Stores 15                        | Saves `y`                                       |
| `00007ff6\`882a14bf`—`mov eax,[rbp-4]`   | `EAX = 20`                       | Loads `a`                                       |
| `00007ff6\`882a14c2`—`imul eax,[rbp-8]`  | `EAX = 100`                      | Calculates `z`                                  |
| `00007ff6\`882a14c6`—`mov [rbp-14h],eax` | Stores 100                       | Saves `z`                                       |
| `00007ff6\`882a14c9`—`mov edx,[rbp-0Ch]` | `EDX = 25`                       | Loads `x`                                       |
| `00007ff6\`882a14cc`—`mov eax,[rbp-10h]` | `EAX = 15`                       | Loads `y`                                       |
| `00007ff6\`882a14cf`—`add edx,eax`       | `EDX = 40`                       | Calculates `x + y`                              |
| `00007ff6\`882a14d1`—`mov eax,[rbp-14h]` | `EAX = 100`                      | Loads `z`                                       |
| `00007ff6\`882a14d4`—`add eax,edx`       | `EAX = 140`                      | Calculates `x + y + z`                          |
| `00007ff6\`882a14d6`—`mov [rbp-18h],eax` | Stores 140                       | Saves `result`                                  |
| `00007ff6\`882a14d9`—`mov eax,[rbp-18h]` | `EAX = 140`                      | Loads result for `printf`                       |
| `00007ff6\`882a14dc`—`lea rcx,[...]`     | `RCX` gets format-string address | First `printf` argument          "Result = %d." |
| `00007ff6\`882a14e3`—`mov edx,eax`       | `EDX = 140`                      | Second `printf` argument                        |
| `00007ff6\`882a14e5`—`call printf`       | Calls output function            | Prints the result                               |
| `00007ff6\`882a14ea`—`mov eax,0`         | `EAX = 0`                        | Sets return value                               |
| `00007ff6\`882a14ef`—`add rsp,40h`       | Restores stack pointer           | Releases stack allocation                       |
| `00007ff6\`882a14f3`—`pop rbp`           | Restores frame pointer           | Function epilogue                               |
| `00007ff6\`882a14f4`—`ret`               | Returns to caller                | Ends `main`                                     |

## 6. C Reconstruction

The following is my reconstructed source, written without relying on the original variable names.

```
#include <stdio.h>

int main(void)
{
    int a = 20;
    int b = 5;

    int c = a + b;
    int d = a - b;
    int e = a * b;

    int result = c + d + e;

    printf("Result = %d\n", result);

    return 0;
}
```

### Differences from the original

- The original uses `x`, `y`, and `z`; the reconstruction uses `c`, `d`, and `e`.
    
- The original uses `int main(void)`; the reconstruction now does too.
    
- The original prints a newline with `\n`; the corrected reconstruction does the same.
    
- The original and reconstruction produce the same output and return value for this program.
    

Variable names do not have to match for a reconstruction to be behaviorally equivalent.

## 7. Verification

### Does it compile?

The initial reconstruction did not compile because the `printf` statement was missing a semicolon. The corrected version should compile.

### Does it behave like the original?

Expected behavior:

```
Result = 140
```

Expected process exit status:

```
0
```

### Test procedure

Compile the reconstruction:

```
gcc -O0 -g -o reconstructed.exe reconstructed.c
```

Run it:

```
.\reconstructed.exe
```

Check the exit status:

```
$LASTEXITCODE
```

### What differences did I discover?

1. The variable names can differ without changing behavior.
    
2. A missing newline changes formatting but not the integer result.
    
3. A missing semicolon causes a compilation error.
    
4. The calculated value `140` is passed to `printf`.
    
5. `main` returns `0`, not `140`.
    
6. The assembly exposes how arithmetic values flow through registers and stack memory.
    

## 8. What I Misunderstood

- **Assuming the assembly must resemble the C syntax:** The compiler can choose a different register order and instruction sequence while preserving behavior.

## 9. Key RE Lesson

Reverse-engineering arithmetic is primarily a data-flow problem. I identify values loaded into registers, follow arithmetic instructions, and trace where results are stored. I then reconstruct the C expressions from those relationships instead of relying on variable names or instruction order alone. Finally, I distinguish between values passed to functions and the function's own return value.

## 10. Checkpoint

- I can explain every relevant instruction in this disassembly.
    
- I can map the stack slots to the correct variables and values.
    
- I can reconstruct addition, subtraction, and multiplication independently.
    
- I can explain how `RCX` and `RDX` pass arguments to `printf` on Windows x64.
    
- I can explain why the program prints `140` but returns `0`.
    
- I compiled and tested the reconstruction independently.
    
- I can reconstruct a modified arithmetic program without looking at the original.
