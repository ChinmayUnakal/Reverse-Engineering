# Program 01 — Simple Function

## 1. Objective

Understand how functions are called, how arguments and return values are passed through registers, and how to identify function boundaries in assembly.

## 2. Original C

```
#include <stdio.h>

int calculate(void)
{
    return 42;
}

int main(void)
{
    int result = calculate();

    printf("Result: %d\n", result);

    return 0;
}
```

## 3. Behavioral prediction

- **Expected output:** `Result: 42`
    
- **Expected return value:** `0` from `main`.
    
- **Important variable values:**
    
    - `calculate()` returns `42`.
        
    - `result = 42`.
        
- **Expected branches or function calls:**
    
    - `main()` calls `calculate()`.
        
    - `main()` calls `printf()` with the format string and result.
        
    - No conditional branches.
        

## 4. Compilation

- **Compiler/version:** GCC 15.2.0 (MSYS2).
    
- **Architecture:** x86-64 Windows.
    
- **Build configuration:** `-O0 -g`.
    
- **Compilation command:**
    

```
gcc -O0 -g -o 01-sim_if.exe 01-Simple_if.c
```

## 5. Assembly observations

|Instruction/address|Register or memory effect|My interpretation|
|---|---|---|
|`1494: mov eax, 2Ah`|Places `42` in `eax`|`calculate()` prepares its return value.|
|`1499: pop rbp`|Restores the previous frame pointer|Function epilogue.|
|`149a: ret`|Returns to the caller|Completes `calculate()`.|
|`14a8: call calculate`|Calls the `calculate()` function|Transfers execution to the function.|
|`14ad: mov [rbp-4], eax`|Stores the returned value in a local variable|Assigns the return value to `result`.|
|`14b0: mov eax, [rbp-4]`|Loads `result` into `eax`|Prepares the second argument for `printf`.|
|`14b3: lea rcx, [7FF7E2864000h]`|Loads the address of a string into `rcx`|First argument to `printf`: format string.|
|`14ba: mov edx, eax`|Copies `result` into `edx`|Second argument to `printf`.|
|`14bc: call printf`|Calls `printf()`|Prints the formatted result.|
|`14c1: mov eax, 0`|Places `0` in `eax`|Sets `main()`'s return value.|
|`14cb: ret`|Returns to the caller|Completes `main()`.|

**Important observations:**

- `0x2A` is hexadecimal for decimal `42`.
    
- On the Windows x64 calling convention, the first integer/pointer arguments are passed through `RCX`, `RDX`, `R8`, and `R9`.
    
- Integer return values are passed through `EAX`.
    
- `calculate()` returns `42` through `EAX`, which `main()` stores in its local variable.
    

**How I identified the format string:**

At `14b3`, the instruction:

```
lea rcx, [7FF7E2864000h]
```

loads the address `0x7FF7E2864000` into `RCX`.

I inspected the memory at that address and discovered the string:

```
Result: %d\n
```

This revealed the exact format string used by `printf()`, even though the string was not directly visible in the assembly instruction.

## 6. C reconstruction

My reconstruction:

```
calculate()
{
    return 42;
}

int main()
{
    int a = calculate();

    printf("Result: %d", a);

    return 0;
}
```

**Interpretation:**

- `calculate()` returns `42`.
    
- The return value is stored in `a`.
    
- `printf()` receives the format string and `a`.
    
- `main()` returns `0`.
    

**Small differences:**

- My `calculate()` is missing its explicit return type. Use `int calculate(void)` for a proper modern C definition.
    
- My `printf()` format string omits `\n`, so the output has no trailing newline. The printed text itself is the same.
    

## 7. Verification

- **Does it compile?** Add `int` as the return type of `calculate()` for standard modern C compilation.
    
- **Does it behave like the original?** Yes, after that correction. Both print `Result: 42`.
    
- **Which test cases did I try?** Called `calculate()` from `main()` and inspected the return value and string argument.
    
- **What differences did I discover?** The original includes a newline in the format string; my reconstruction does not.

## 8. Remember

- The return value of a function is not necessarily stored in memory first; here it is returned through `EAX`.
    
- `lea` loads an address, not the string contents themselves.
    
- Inspecting the memory at that address can reveal strings that are not obvious from the assembly listing.
    
- The Windows x64 calling convention determines which registers contain function arguments.
    

## 9. Key RE lesson

A function returning an integer commonly places its return value in `EAX` on x86-64 Windows. The caller can then store or use that value. Function arguments are passed through designated registers, and inspecting memory addresses loaded into those registers can reveal strings and other data used by the program. This is useful when reconstructing function calls without access to the original source.

## 10. Checkpoint

- I can explain every relevant instruction.
    
- I can reconstruct the program without the source.
    
- I can explain how `calculate()` returns `42` through `EAX`.
    
- I can explain how I recovered the format string by inspecting memory.
    
- I can solve a slightly modified version independently.