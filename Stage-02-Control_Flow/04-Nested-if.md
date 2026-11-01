# Program 04 — Nested If

## 1. Objective

Understand how nested `if` statements translate into sequential comparisons and conditional jumps.

## 2. Original C

```
int main(void)
{
    int a = 10;
    int b = 5;

    if (a > 0) {
        if (b > 0)
            return 1;
    }

    return 0;
}
```

## 3. Behavioral Prediction

- **Expected output:** None.
    
- **Expected return value:** `1`.
    
- **Variables:** `a = 10`, `b = 5`.
    
- **Control flow:** Both conditions are true, so return `1`; otherwise, return `0`.
    

## 4. Compilation

- **Compiler/version:** GCC 15.2.0 (MSYS2).
    
- **Architecture:** x86-64 Windows.
    
- **Build configuration:** `-O0 -g`.
    
- **Compilation command:**
    

```
gcc -O0 -g -o 04_nested_if.exe 04_nested_if.c
```

## 5. Assembly Observations

|Instruction|Effect|Interpretation|
|---|---|---|
|`cmp [rbp-4], 0`|Compares `a` with `0`|Checks `a > 0`.|
|`jle main+0x2e`|Jumps if `a <= 0` (signed)|Skips the inner `if` if the outer condition fails.|
|`cmp [rbp-8], 0`|Compares `b` with `0`|Checks `b > 0`.|
|`jle main+0x2e`|Jumps if `b <= 0` (signed)|Skips the success return if the inner condition fails.|
|`mov eax, 1`|Sets `EAX = 1`|Both conditions passed.|
|`jmp main+0x33`|Jumps past the failure return|Continues to the function epilogue.|
|`mov eax, 0`|Sets `EAX = 0`|At least one condition failed.|
|`ret`|Returns to the caller|Returns the value in `EAX`.|

**Key observation:** The two `jle` instructions target the same failure path. The inner condition is reached only if the outer condition succeeds.

## 6. C Reconstruction

**My reconstruction:**

```
int main()
{
    int a = 10;
    int b = 5;

    if(a > 0)
    {
        if(b > 0)
        {
            return 1;
        }
    }

    return 0;
}
```

### Evaluation

- **Logic:** Correct.
    
- **Expected return value:** `1`.
    
- **Differences:** Only formatting and braces; the control flow matches the original.
    
- **Equivalence:** Returns `1` when both `a > 0` and `b > 0`; otherwise, returns `0`.
    

## 7. Verification

- **Does it compile?** Not yet verified.
    
- **Expected return value:** `1`.
    
- **Test cases:**
    

|`a`|`b`|Expected return|
|---|---|---|
|10|5|1|
|0|5|0|
|10|0|0|
|-1|5|0|

## 8. What I Misunderstood

No reconstruction mistake identified.

## 9. Key RE Lesson

Nested `if` statements create layered control flow: the inner condition is evaluated only when the outer condition succeeds. Conditional jumps can skip directly to a shared failure path, rather than requiring a separate failure block for every condition. Recognizing these shared destinations helps reconstruct nested logic from assembly.

## 10. Checkpoint

- Identify both comparisons and conditional jumps.
    
- Explain how the outer condition controls the inner condition.
    
- Reconstruct the program's logic.
    
- Explain why my reconstruction is equivalent.
    
- Compile and verify the program.
    
- Test modified values that cause each condition to fail independently.