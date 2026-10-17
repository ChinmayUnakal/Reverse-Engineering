# Program 13 — Signed Comparison

## 1. Objective

Learn how signed integer comparisons are implemented in x86-64 assembly. Understand how `CMP` sets CPU flags and how a conditional jump such as `JGE` can implement a C condition like `a < b`.

## 2. Original C

```
int main(void)
{
    int a = -5;
    int b = 3;

    if (a < b)
        return 1;

    return 0;
}
```

## 3. Behavioral prediction

- **Expected output:** None.
    
- **Expected return value:** `1`.
    
- **Important variable values:**
    
    - `a = -5` → 32-bit representation `0xFFFFFFFB`
        
    - `b = 3` → 32-bit representation `0x00000003`
        
    - `a < b` → true
        
    - Return value → `1`
        
- **Expected branches or function calls:**
    
    - GCC's `__main` runtime helper is called.
        
    - The program compares `a` and `b`.
        
    - Since `a < b`, execution takes the path that returns `1`.
        
    - The branch that returns `0` is skipped.
        

## 4. Compilation

- **Compiler/version:** GCC 15.2.0 (MSYS2).
    
- **Architecture:** x86-64 Windows.
    
- **Build configuration:** `-O0 -g`.
    
- **Compilation command:**
    

```
gcc -O0 -g -o sign_comp.exe 13_signed_comparison.c
```

## 5. Assembly observations

|Instruction/address|Register or memory effect|My interpretation|
|---|---|---|
|`push rbp`|Saves the previous frame pointer|Function prologue|
|`mov rbp, rsp`|Establishes the stack frame|Sets the frame pointer|
|`sub rsp, 30h`|Reserves stack space|Allocates the stack frame|
|`call __main`|Calls GCC's runtime helper|Compiler-generated initialization|
|`mov dword ptr [rbp-4], 0FFFFFFFBh`|Stores `0xFFFFFFFB`|Initializes `a = -5`|
|`mov dword ptr [rbp-8], 3`|Stores `3`|Initializes `b = 3`|
|`mov eax, dword ptr [rbp-4]`|Loads `a` into `EAX`|Prepares the first operand|
|`cmp eax, dword ptr [rbp-8]`|Compares `a` against `b` by setting flags|Tests the relationship between the signed integers|
|`jge main+0x2a`|Jumps if `a >= b` using signed comparison|Skips `return 1` when the condition `a < b` is false|
|`mov eax, 1`|Places `1` in `EAX`|Implements `return 1`|
|`jmp main+0x2f`|Jumps to the epilogue|Skips the `return 0` path|
|`mov eax, 0`|Places `0` in `EAX`|Implements `return 0`|
|`add rsp, 30h`|Releases stack space|Function epilogue|
|`pop rbp`|Restores the previous frame pointer|Function epilogue|
|`ret`|Returns to the caller|Returns the value in `EAX`|

### Understanding the key instructions

**1.** `CMP`

```
cmp eax, dword ptr [rbp-8]
```

Conceptually, the CPU evaluates:

```
EAX - [RBP-8]
-5 - 3 = -8
```

`CMP` does not save the subtraction result. It updates the CPU flags based on the comparison.

**2.** `JGE`

```
jge main+0x2a
```

`JGE` means **jump if greater than or equal, using signed comparison**. It jumps when the first operand is greater than or equal to the second.

Here, `-5 >= 3` is false, so the jump is not taken. Execution continues to `mov eax, 1`.

### Control flow

```
Compare a with b
       |
       v
    a >= b?
    /     \
  Yes      No
   |        |
return 0  return 1
```

Notice that the assembly tests the _opposite_ condition from the source-level `if (a < b)`. This is a common compiler strategy: jump over the `if` body when its condition is false.

## 6. C reconstruction

```
int main(void)
{
    int a = -5;
    int b = 3;

    if (a < b)
        return 1;

    return 0;
}
```

The compiler could also implement the same logic using an equivalent condition such as `if (a >= b) return 0; return 1;`. What matters is that the resulting control flow and return values match.

## 7. Verification

- **Does it compile?** Compile with the command in Section 4.
    
- **Does it behave like the original?** Yes, if it returns `1` for the given values.
    
- **Which test cases did I try?** Record the cases you actually ran.
    
- **What differences did I discover?** The assembly uses `JGE` to skip the `return 1` path when `a >= b`, rather than directly using a jump for `a < b`.
    

Useful test cases:

|`a`|`b`|`a < b`|Expected return|
|---|---|---|---|
|`-5`|`3`|True|`1`|
|`3`|`3`|False|`0`|
|`5`|`3`|False|`0`|
|`-10`|`-2`|True|`1`|

To test the process exit code in PowerShell:

```
.\sign_comp.exe
$LASTEXITCODE
```

For the original values, the expected exit code is `1`.

## 8. What I misunderstood

No reconstruction mistake identified.
## 9. Key RE lesson

`CMP` updates CPU flags, and conditional jumps use those flags to control execution. Signed comparisons use conditions such as `JL` and `JGE`, whereas unsigned comparisons use conditions such as `JB` and `JAE`. When reconstructing C, follow the branch destination and determine which path returns each value instead of translating the jump mnemonic in isolation.

## 10. Checkpoint

- I can explain what `CMP` changes and what it does not change.
    
- I can explain why `JGE` is not taken for `-5` and `3`.
    
- I can distinguish signed `JGE` from unsigned `JAE`.
    
- I can reconstruct the `if` statement by tracing both paths.
    
- I can independently solve a modified example involving negative numbers and equality.