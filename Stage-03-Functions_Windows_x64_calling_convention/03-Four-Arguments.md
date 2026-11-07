# Program 03 — Four Arguments

## 1. Objective

Understand how Windows x64 passes four function arguments through registers and how arithmetic operations are reconstructed from assembly.

## 2. Original C

```
#include <stdio.h>

int calculate(int a, int b, int c, int d)
{
    return a + b * 2 + c * 3 + d * 4;
}

int main(void)
{
    int result = calculate(10, 20, 30, 40);
    printf("Result: %d\n", result);
    return 0;
}
```

## 3. Behavioral prediction

- **Expected output:** `Result: 300`
    
- **Expected return value:** `calculate()` returns 300; `main()` returns 0.
    
- **Important values:** `a = 10`, `b = 20`, `c = 30`, `d = 40`.
    
- **Expected function calls:** `main()` → `calculate(10, 20, 30, 40)` → `printf()`.
    

## 4. Compilation

- **Compiler/version:** GCC 15.2.0 (MSYS2).
    
- **Architecture:** x86-64 Windows.
    
- **Build configuration:** `-O0 -g`.
    
- **Compilation command:**
    

```
gcc -O0 -g -o 03-four_args.exe 03-four_arguments.c
```

## 5. Assembly observations

|Instruction|Register or memory effect|Interpretation|
|---|---|---|
|`mov [rbp+10h], ecx`|Saves ECX|First argument `a`.|
|`mov [rbp+18h], edx`|Saves EDX|Second argument `b`.|
|`mov [rbp+20h], r8d`|Saves R8D|Third argument `c`.|
|`mov [rbp+28h], r9d`|Saves R9D|Fourth argument `d`.|
|`lea edx, [rax+rax]`|Doubles RAX|Calculates `b * 2`.|
|`lea ecx, [rdx+rax]`|Adds values|Calculates `a + b * 2`.|
|`add eax, eax`|Doubles EAX|First step of `c * 3`.|
|`add eax, edx`|Adds c again|Completes `c * 3`.|
|`shl eax, 2`|Shifts left by 2 bits|Calculates `d * 4`.|
|`add eax, edx`|Adds partial results|Produces final result (300).|
|`mov r9d, 28h`|R9D = 40|Prepares fourth argument.|
|`mov r8d, 1Eh`|R8D = 30|Prepares third argument.|
|`mov edx, 14h`|EDX = 20|Prepares second argument.|
|`mov ecx, 0Ah`|ECX = 10|Prepares first argument.|
|`call calculate`|Calls function|Executes calculation.|

## 6. C reconstruction

```
int calculate(int a, int b, int c, int d)
{
    return b + b + a + c + c + c + (d << 2);
}

int main(void)
{
    int a = calculate(10, 20, 30, 40);

    printf("Result: %d", a);

    return 0;
}
```

## 7. Verification

- **Does it compile?** Yes.
    
- **Does it behave like the original?** Yes.
    
- **Test case:** `calculate(10, 20, 30, 40)` returns 300.
    
- **Differences:** Arithmetic expression is reconstructed differently, but the result is equivalent for these inputs. The `printf()` newline is omitted.
    

## 8. What I misunderstood

- No major reconstruction errors.
    
- `lea` can perform arithmetic, not just calculate memory addresses.
    
- `shl eax, 2` shifts left by two bits, equivalent to multiplying by 4 for this integer operation.
    
- `b + b` represents`b * 2`, while `c + c + c`represents`c * 3` and `(d << 2)` represents`d * 4.`
    

## 9. Key RE lesson

Windows x64 passes the first four integer arguments through RCX, RDX, R8, and R9. Assembly can implement multiplication using repeated addition, `lea`, or bit shifting instead of explicit multiplication instructions. Recognizing these equivalent operations helps reconstruct the original C expression.

## 10. Checkpoint

- I can identify all four argument registers.
    
- I can explain the arithmetic instructions.
    
- I can reconstruct the calculation.
    
- I can explain why my reconstruction is equivalent.
    
- I can independently reconstruct a modified four-argument function.


### Mistakes and concepts worth understanding

There are three things to pay attention to:

1. Reconstruction is correct, but not necessarily identical for every possible integer input.

For normal, non-overflowing integer values:

\[ a+2b+3c+4d = 2b+a+3c+(d << 2) \]

However, signed integer overflow in C is undefined behavior, so don't assume every algebraically equivalent expression is interchangeable for all possible inputs.

2. Understand `lea`.

```
lea edx, [rax+rax]
```

This calculates `rax + rax` without accessing memory. Here, GCC uses it to multiply by 2.

3. Understand `shl`.

```
shl eax, 2
```

Shifts the bits left by two positions. For this operation, it multiplies the value by 4.