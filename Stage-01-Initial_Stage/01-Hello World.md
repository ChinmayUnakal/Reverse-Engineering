# 01 — Hello World: Function Calls, Arguments, and Return Values

## 1. Objective

Understand how a simple C program translates into Windows x64 assembly.

Skills:

- Recognize a function prologue and epilogue.
    
- Understand how a string is stored in memory.
    
- Track a function argument through registers.
    
- Understand `call`, `jmp`, and `ret`.
    
- Recognize an imported library function and a jump stub.
    
- Distinguish program logic from compiler-generated runtime code.
    

## 2. Original C

```c
#include <stdio.h>

int main(void)
{
    printf("Hello, World!\n");
    return 0;
}
```

## 3. Behavioral prediction

- **Expected output:** `Hello, World!` followed by a newline.
    
- **Expected return value:** `0` from `main`.
    
- **Important variable values:** No local variables.
    
- **Expected branches or function calls:** `main` calls a library function to print the string.
    
- **Expected memory:** A string literal stored in the executable's read-only data section.
    

## 4. Compilation

- **Compiler:** GCC (record the exact version if known).
    
- **Architecture:** Windows x64.
    
- **Build configuration:** `-O0 -g`.
    
- **Compilation command:**
    

```bash
gcc -O0 -g -o hello.exe 01_hello_world.c
```

- **Debugger:** WinDbg.
    
- **Important caveat:** `-O0` disables most optimization passes, but it does not guarantee that every source-level operation maps directly to a distinct assembly instruction.
    

## 5. Assembly observations

### A. Function prologue

```asm
push rbp
mov  rbp, rsp
sub  rsp, 20h
```

|Instruction|Effect|Interpretation|
|---|---|---|
|`push rbp`|Saves the previous `RBP` on the stack.|Preserves the caller's frame pointer.|
|`mov rbp, rsp`|Copies `RSP` into `RBP`.|Establishes a stack-frame reference point.|
|`sub rsp, 20h`|Decreases `RSP` by 32 bytes.|Reserves stack space and provides the required call-area/alignment setup for this function.|

**Important:** `20h` is hexadecimal for 32 decimal. The exact stack-frame layout depends on the compiler and calling convention.

### B. GCC runtime initialization

```asm
call hello!__main
```

- This calls a GCC-generated/runtime helper before the main program's printing operation.
    
- `__main` is not the C `main` function.
    
- Its presence is toolchain-specific; it is not required in every C program.
    

### C. Loading the string address

```asm
lea rax, [7FF64D4F4000h]
mov rcx, rax
```

- `lea` loads the effective address of the string into `RAX`; it does not load the string's characters into the register.
    
- `mov rcx, rax` copies that address into `RCX`.
    
- Under the Windows x64 calling convention, `RCX` carries the first integer or pointer argument.
    

### D. Calling the printing function

```asm
call 00007FF64D4F28D8
```

The call initially targets a jump stub in the executable.

The stub contains:

```asm
jmp qword ptr [hello+0x83c8]
```

The pointer stored at `hello+0x83c8` resolves to:

```text
ucrtbase!puts
```

**Control flow:**

1. `main` executes `call` to the stub.
    
2. The stub executes an indirect `jmp` to `puts`.
    
3. `puts` prints the string and appends a newline.
    
4. `puts` returns to the instruction immediately after the original `call` in `main`.
    

The stub redirects execution; it does not perform the printing itself.

### E. Why `puts` instead of `printf`?

My original C source uses:

```c
printf("Hello, World!\n");
```

However, the observed call resolves to `ucrtbase!puts`, and the string in memory is:

```text
48 65 6c 6c 6f 2c 20 57 6f 72 6c 64 21 00
```

These bytes represent `Hello, World!` followed by a NUL terminator. There is no newline byte in the literal.

`puts` automatically appends a newline, so this is consistent with the observed behavior. The compiler may have transformed the original `printf` call into `puts`; I should verify this by inspecting the exact compiler version, flags, and generated code rather than assuming every build does the same thing.

### F. Function epilogue and return

```asm
mov eax, 0
add rsp, 20h
pop rbp
ret
```

|Instruction|Effect|Interpretation|
|---|---|---|
|`mov eax, 0`|Sets `EAX` to zero.|Sets the return value of `main` to `0`.|
|`add rsp, 20h`|Adds 32 to `RSP`.|Releases the stack space reserved by the prologue.|
|`pop rbp`|Restores the saved `RBP`.|Restores the caller's frame pointer.|
|`ret`|Returns to the saved return address.|Transfers control back to the caller.|

## 6. C reconstruction

Reconstructed without consulting the original:

```c
#include <stdio.h>

int main(void)
{
    puts("Hello, World!");
    return 0;
}
```

This is behaviorally equivalent for the observed output and return status, assuming ordinary execution with a working standard output stream. It is not necessarily identical to the original source or its exact library-call behavior in every possible environment.

## 7. Verification

-  Compile the reconstructed source.
    
-  Confirm that it prints `Hello, World!` followed by a newline.
    
-  Confirm that `main` returns `0`.
    
-  Inspect the call target using WinDbg.
    
-  Follow the jump stub to the resolved function.
    
-  Compare the original and reconstructed binaries without expecting identical assembly.
    

**Test cases:**

- Normal execution.
    
- Output redirected to a file.
    
- Compare behavior when the original uses `printf` and the reconstruction uses `puts`.
    

## 8. What I misunderstood

- Initially, I expected the assembly to call `printf` directly because that is what the source code contains.
    
- I initially treated the jump stub as though it might be the actual printing function.
    
- I needed to follow the indirect jump and resolve its destination to identify `puts`.
    
- I learned that assembly reflects compiler and linker decisions, not just a literal translation of each C statement.
    

## 9. Key RE lesson

A reverse engineer must follow data and control flow rather than relying only on source-level expectations. Here, the string's address is loaded into a register, passed using the Windows x64 calling convention, and eventually consumed by `puts` through a jump stub. The compiler-generated prologue, runtime initialization, and epilogue are part of the binary, but they are not the core printing logic.

## 10. Checkpoint

-  I can explain the prologue and epilogue.
    
-  I can explain why `lea` obtains the string address.
    
-  I can explain why the first argument is placed in `RCX`.
    
-  I can distinguish `call`, `jmp`, and `ret`.
    
-  I can follow a jump stub to its destination.
    
-  I can reconstruct equivalent C without looking at the source.
    
-  I can explain why the assembly may call `puts` even though the source calls `printf`.
    
-  I can repeat the analysis on a slightly modified program without step-by-step guidance.