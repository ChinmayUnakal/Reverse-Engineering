# Program 04 — More Than Four Arguments

## 1. Objective

Understand how Windows x64 passes arguments beyond the first four through the stack and how the called function accesses them.

## 2. Original C

```
#include <stdio.h>

int calculate(int a, int b, int c, int d, int e, int f)
{
    return a + b + c + d + e + f;
}

int main(void)
{
    int result = calculate(10, 20, 30, 40, 50, 60);
    printf("Result: %d\n", result);
    return 0;
}
```

## 3. Behavioral prediction

- **Expected output:** `Result: 210`
    
- **Expected return value:** `calculate()` returns 210; `main()` returns 0.
    
- **Important values:** `a = 10`, `b = 20`, `c = 30`, `d = 40`, `e = 50`, `f = 60`.
    
- **Expected function calls:** `main()` → `calculate(10, 20, 30, 40, 50, 60)` → `printf()`.
    

## 4. Compilation

- **Compiler/version:** GCC 15.2.0 (MSYS2).
    
- **Architecture:** x86-64 Windows.
    
- **Build configuration:** `-O0 -g`.
    
- **Compilation command:**
    

```
gcc -O0 -g -o 04-morethan4.exe 04-more-than-4-args.c
```

## 5. Assembly observations

|Instruction|Register or memory effect|Interpretation|
|---|---|---|
|`mov [rbp+10h], ecx`|Saves ECX|First argument `a`.|
|`mov [rbp+18h], edx`|Saves EDX|Second argument `b`.|
|`mov [rbp+20h], r8d`|Saves R8D|Third argument `c`.|
|`mov [rbp+28h], r9d`|Saves R9D|Fourth argument `d`.|
|`mov eax, [rbp+30h]`|Loads fifth argument|Retrieves `e = 50`.|
|`mov eax, [rbp+38h]`|Loads sixth argument|Retrieves `f = 60`.|
|`add edx, eax`|Adds values|Accumulates the arguments.|
|`mov [rsp+20h], 32h`|Stores 50|Prepares fifth argument.|
|`mov [rsp+28h], 3Ch`|Stores 60|Prepares sixth argument.|
|`mov r9d, 28h`|R9D = 40|Prepares fourth argument.|
|`mov r8d, 1Eh`|R8D = 30|Prepares third argument.|
|`mov edx, 14h`|EDX = 20|Prepares second argument.|
|`mov ecx, 0Ah`|ECX = 10|Prepares first argument.|
|`call calculate`|Calls function|Executes calculation.|
|`ret`|Returns through EAX|Returns 210.|

## 6. C reconstruction

```
int calculate(int a, int b, int c, int d, int e, int f)
{
    return a + b + c + d + e + f;
}

int main(void)
{
    int result = calculate(10, 20, 30, 40, 50, 60);

    printf("Result: %d", result);

    return 0;
}
```

## 7. Verification

- **Does it compile?** Expected to compile successfully.
    
- **Does it behave like the original?** Yes, logically equivalent.
    
- **Test case:** `calculate(10, 20, 30, 40, 50, 60)` returns 210.
    
- **Differences:** Only the newline in `printf()` is omitted.
    

## 8.Remember
    
- Arguments beyond the first four are passed through stack memory.
    
- The caller prepares stack arguments before executing `call`.
    
- The callee accesses these arguments at different offsets relative to its frame pointer.
    

## 9. Key RE lesson

Windows x64 passes the first four integer arguments through RCX, RDX, R8, and R9. Additional arguments are passed through the stack, after the 32-byte shadow space. In this example, the fifth and sixth arguments are stored at `[rsp+20h]` and `[rsp+28h]` by the caller and accessed at `[rbp+30h]` and `[rbp+38h]` by the callee. Recognizing these locations helps reconstruct functions with more than four arguments.

## 10. Checkpoint

- I can identify the first four argument registers.
    
- I can identify the fifth and sixth arguments on the stack.
    
- I can reconstruct the addition.
    
- I can explain why my reconstruction is equivalent.
