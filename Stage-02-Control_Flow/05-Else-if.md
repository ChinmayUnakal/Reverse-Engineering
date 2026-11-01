# Program 05 — Else If

## 1. Objective

Understand how an `if-else if-else` chain becomes sequential comparisons, conditional jumps, and different return values in x86-64 assembly.

## 2. Original C

```
int main(void)
{
    int value = 15;

    if (value < 10)
        return 1;
    else if (value < 20)
        return 2;
    else
        return 3;
}
```

## 3. Behavioral Prediction

- **Expected output:** None.
    
- **Expected return value:** `2`.
    
- **Important variables:** `value = 15`.
    
- **Expected control flow:** `value < 10` is false, so the second condition is checked. Since `15 < 20` is true, the program returns `2`.
    

## 4. Compilation

- **Compiler/version:** GCC 15.2.0 (MSYS2).
    
- **Architecture:** x86-64 Windows.
    
- **Build configuration:** `-O0 -g`.
    
- **Compilation command:**
    

```
gcc -O0 -g -o 05_else_if.exe 05_else_if.c
```

## 5. Assembly Observations

|Instruction/address|Register or memory effect|Interpretation|
|---|---|---|
|`mov [rbp-4], 0Fh`|Stores `15` in the local variable|Initializes `value`.|
|`cmp [rbp-4], 9`|Compares `value` with `9`|Checks the boundary for `value < 10`.|
|`jg main+0x21`|Jumps if `value > 9` (signed)|Skips the first return and checks the next condition.|
|`mov eax, 1`|Sets `EAX = 1`|Returns `1` when `value <= 9`.|
|`jmp main+0x33`|Jumps to the epilogue|Prevents execution from falling through into another branch.|
|`cmp [rbp-4], 13h`|Compares `value` with `19`|Checks the boundary for `value < 20`.|
|`jg main+0x2e`|Jumps if `value > 19` (signed)|Skips the second return and selects the final `else` result.|
|`mov eax, 2`|Sets `EAX = 2`|Returns `2` when `10 <= value <= 19`.|
|`jmp main+0x33`|Jumps to the epilogue|Skips the final `else` result.|
|`mov eax, 3`|Sets `EAX = 3`|Returns `3` when `value >= 20`.|
|`ret`|Returns to the caller|Returns the value in `EAX`.|

**Key observation:** The `jg` instructions jump when a condition's upper boundary has been exceeded. For `value = 15`, the first `jg` is taken, but the second is not. Execution therefore reaches `mov eax, 2`.

## 6. C Reconstruction

**My reconstruction (preserved exactly):**

```
int main()
{
    int a = 15;

    if(a <= 9)
    {
        return 1;
    }
    else if(a <= 19)
    {
        return 2;
    }

    return 3;
}
```

### Evaluation

- **Logic:** Correct.
    
- **Expected return value:** `2`.
    
- **Differences:** The original uses `value < 10` and `value < 20`; my reconstruction uses `a <= 9` and `a <= 19`.
    
- **Equivalence:** Because `a` is an integer, `a < 10` is equivalent to `a <= 9`, and `a < 20` is equivalent to `a <= 19`. The final `return 3` also reproduces the `else` behavior, even without an explicit `else` keyword.
    

## 7. Verification

- **Does it compile?** Not yet verified.
    
- **Does it behave like the original?** Expected to, based on the control-flow analysis.
    
- **Test cases:**
    

|`value`|Expected return|
|---|---|
|`5`|`1`|
|`15`|`2`|
|`25`|`3`|
|`10`|`2`|
|`20`|`3`|

Run in PowerShell:

```
.\05_else_if.exe
$LASTEXITCODE
```

To test different values, modify the initializer, recompile, and run the program again.

## 8. What I Misunderstood

No reconstruction mistake identified. The important detail is that comparisons against `9` and `19` implement the same integer boundaries as comparisons using `< 10` and `< 20`.

## 9. Key RE Lesson

An `else if` chain can compile into sequential comparisons and conditional jumps. Each branch either selects a return value or transfers execution to the next condition. Comparing against `9` instead of `10` can implement `value < 10` when the variable is an integer. Always consider the comparison instruction, jump condition, and jump destination together to reconstruct the original logic.

## 10. Checkpoint

- Explain how the sequential comparisons implement the `else if` chain.
    
- Identify the return values `1`, `2`, and `3`.
    
- Explain why my reconstruction is equivalent.
    
- Compile and verify the program.
    
- Test values at and around the boundaries: `9`, `10`, `19`, and `20`.
    
- Reconstruct a modified `else if` chain independently.