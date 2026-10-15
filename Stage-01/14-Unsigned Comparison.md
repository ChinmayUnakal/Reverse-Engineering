# Program 14 — Unsigned Comparison

## 1. Objective

Learn to identify unsigned comparisons in x86-64 assembly. Understand how `JAE` and other unsigned conditional jumps differ from signed jumps such as `JGE`, and how signedness changes comparison results.

## 2. Original C

```
int main(void)
{
    unsigned int a = 0xFFFFFFFF;
    unsigned int b = 1;

    if (a > b)
        return 1;

    return 0;
}
```

## 3. Behavioral prediction

- **Expected output:** None.
    
- **Expected return value:** `1`.
    
- **Important variable values:**
    
    - `a = 0xFFFFFFFF` → unsigned decimal value `4294967295`
        
    - `b = 1` → unsigned decimal value `1`
        
    - `a > b` → true
        
    - Return value → `1`
        
- **Expected branches or function calls:**
    
    - Calls GCC's `__main` runtime helper.
        
    - Compares `b` with `a`.
        
    - Uses an unsigned conditional jump to skip the `return 1` path when `b >= a`.
        
    - Since `1` is not greater than or equal to `4294967295` as an unsigned value, the jump is not taken, and the program returns `1`.
        

## 4. Compilation

- **Compiler/version:** GCC 15.2.0 (MSYS2).
    
- **Architecture:** x86-64 Windows.
    
- **Build configuration:** `-O0 -g`.
    
- **Compilation command:**
    

```
gcc -O0 -g -o unsign_comp.exe 14_unsigned_comparison.c
```

## 5. Assembly observations

|Instruction/address|Register or memory effect|My interpretation|
|---|---|---|
|`push rbp`|Saves the previous frame pointer|Function prologue|
|`mov rbp, rsp`|Establishes the stack frame|Sets the frame pointer|
|`sub rsp, 30h`|Reserves stack space|Allocates the stack frame|
|`call __main`|Calls GCC's runtime helper|Compiler-generated initialization|
|`mov dword ptr [rbp-4], 0FFFFFFFFh`|Stores `0xFFFFFFFF`|Initializes unsigned `a` to `4294967295`|
|`mov dword ptr [rbp-8], 1`|Stores `1`|Initializes unsigned `b` to `1`|
|`mov eax, dword ptr [rbp-4]`|Loads `a` into `EAX`|Prepares `a` for comparison|
|`cmp dword ptr [rbp-8], eax`|Compares `b` against `a` by setting flags|Tests the reverse condition, `b >= a`|
|`jae main+0x2a`|Jumps if the first operand is above or equal to the second, unsigned|Skips `return 1` if `b >= a`|
|`mov eax, 1`|Places `1` in `EAX`|Implements `return 1`|
|`jmp main+0x2f`|Jumps to the epilogue|Skips the `return 0` path|
|`mov eax, 0`|Places `0` in `EAX`|Implements `return 0`|
|`add rsp, 30h`|Releases stack space|Function epilogue|
|`pop rbp`|Restores the previous frame pointer|Function epilogue|
|`ret`|Returns to the caller|Returns the value in `EAX`|

### Understanding the comparison

The key instructions are:

```
mov eax, dword ptr [rbp-4]
cmp dword ptr [rbp-8], eax
jae main+0x2a
```

The comparison is effectively:

```
b - a
1 - 4294967295
```

`CMP` does not store this subtraction result; it updates the CPU flags.

`JAE` means **jump if above or equal, using unsigned comparison**. The jump is taken when `b >= a` as unsigned integers.

For this program:

```
b = 1
a = 4294967295

1 >= 4294967295  → false
```

Therefore, the jump is not taken, and execution reaches `mov eax, 1`.

### Signed versus unsigned interpretation

The bit pattern `0xFFFFFFFF` has two common interpretations:

|Type|Decimal interpretation|
|---|---|
|`int` (32-bit signed)|`-1`|
|`unsigned int` (32-bit unsigned)|`4294967295`|

Consequently:

```
(unsigned int)0xFFFFFFFF > 1  // true
(int)0xFFFFFFFF > 1           // false
```

The bits are identical; the comparison's signedness changes the result.

## 6. C reconstruction

**Status: Partially reconstructed.** You recognized the equivalent reverse comparison, `b < a`, but your declarations use signed `int`. That changes the meaning of the values and makes your reconstruction return `0`, not `1`.

Your reconstruction:

```
int main()
{
    int a = -1;
    int b = 1;

    if (b < a)
        return 1;

    return 0;
}
```

Corrected reconstruction:

```
int main(void)
{
    unsigned int a = 0xFFFFFFFF;
    unsigned int b = 1;

    if (b < a)
        return 1;

    return 0;
}
```

This is behaviorally equivalent for the given values because `b < a` is the same comparison as `a > b` when both operands have the same unsigned type.

## 7. Verification

- **Does it compile?** Compile the corrected reconstruction using the command in Section 4.
    
- **Does it behave like the original?** It should return `1`.
    
- **Which test cases did I try?** Record your actual tests after running them.
    
- **What differences did I discover?** Using signed `int` changes `a` from the unsigned value `4294967295` to the signed value `-1`.
    

Useful test cases:

|Unsigned `a`|Unsigned `b`|Is `a > b` true?|Expected return|
|---|---|---|---|
|`4294967295`|`1`|Yes|`1`|
|`1`|`1`|No|`0`|
|`5`|`10`|No|`0`|
|`10`|`5`|Yes|`1`|

To inspect the process exit code in PowerShell:

```
.\unsign_comp.exe
$LASTEXITCODE
```

For this program, the expected exit code is `1`.

## 8. What I misunderstood

I used `int` instead of `unsigned int`. On a typical 32-bit `int` implementation, the bit pattern `0xFFFFFFFF` represents `-1`, so `b < a` becomes `1 < -1`, which is false. The original program interprets the same pattern as `4294967295`, making `1 < a` true.

## 9. Key RE lesson

Unsigned comparisons interpret bit patterns as nonnegative integers, while signed comparisons interpret the highest bit as the sign bit. In assembly, `JAE` is an unsigned condition, whereas `JGE` is a signed condition. Always inspect both the comparison operands and the conditional jump before reconstructing the original C types and logic.

## 10. Checkpoint

- I can explain why `0xFFFFFFFF` represents `4294967295` as an unsigned integer and `-1` as a signed integer.
    
- I can explain the difference between `JAE` and `JGE`.
    
- I can trace why the `JAE` branch is not taken here.
    
- I can reconstruct the reversed comparison without changing signedness.
    
- I can independently solve a comparison where the high bit is set.