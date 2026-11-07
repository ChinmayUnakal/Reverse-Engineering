# Program 02 — Two Arguments

## 1. Objective

Understand how Windows x64 passes multiple function arguments through registers, how functions access them, and how integer return values are handled.

## 2. Original C

```
#include <stdio.h>

int add(int a, int b)
{
    return a + b;
}

int main(void)
{
    int result = add(10, 20);
    printf("Result: %d\n", result);
    return 0;
}
```

## 3. Behavioral prediction

- **Expected output:** `Result: 30`
    
- **Expected return value:** `main()` returns 0; `add()` returns 30.
    
- **Important variable values:** `a = 10`, `b = 20`, `result = 30`.
    
- **Expected function calls:** `main()` → `add(10, 20)` → `printf()`.
    

## 4. Compilation

- **Compiler/version:** GCC 15.2.0 (MSYS2).
    
- **Architecture:** x86-64 Windows.
    
- **Build configuration:** `-O0 -g`.
    
- **Compilation command:**
    

```
gcc -O0 -g -o 02-two_args.exe 02-Two_arguments.c
```

## 5. Assembly observations

|Instruction|Register or memory effect|Interpretation|
|---|---|---|
|`mov [rbp+10h], ecx`|Stores ECX|Saves first argument (`a = 10`).|
|`mov [rbp+18h], edx`|Stores EDX|Saves second argument (`b = 20`).|
|`mov edx, [rbp+10h]`|Loads first argument|Retrieves `a`.|
|`mov eax, [rbp+18h]`|Loads second argument|Retrieves `b`.|
|`add eax, edx`|Adds both values|Calculates `a + b = 30`.|
|`ret`|Returns through EAX|Returns 30 to the caller.|
|`mov edx, 14h`|EDX = 20|Prepares second argument.|
|`mov ecx, 0Ah`|ECX = 10|Prepares first argument.|
|`call add`|Calls add function|Executes the addition.|
|`mov [rbp-4], eax`|Saves EAX|Stores the returned result (30).|

## 6. C reconstruction

```
int add(int a, int b)
{
    return a + b;
}

int main(void)
{
    int a = add(10, 20);

    printf("Result: %d", a);

    return 0;
}
```

## 7. Verification

- **Does it compile?** Yes.
    
- **Does it behave like the original?** Yes.
    
- **Test case:** `add(10, 20)` returns 30.
    
- **Differences:** Minor formatting difference in the `printf()` newline. No functional difference.
    

## 8. Remember

- Windows x64 passes the first two integer arguments using `ECX` and `EDX`.
    
- The function's return value is placed in `EAX`.
    

## 9. Key RE lesson

Under the Windows x64 calling convention, the first four integer or pointer arguments are passed through `RCX`, `RDX`, `R8`, and `R9` (32-bit values use their corresponding registers such as ECX and EDX). Here, `add()` receives 10 in ECX and 20 in EDX. It adds them and returns 30 through EAX. Recognizing argument registers and return registers helps reconstruct function calls from assembly.

## 10. Checkpoint

- I can explain the argument-passing instructions.
    
- I can identify the return value in EAX.
    
- I can reconstruct the function and its caller.
    
- I can reconstruct a modified two-argument function independently.