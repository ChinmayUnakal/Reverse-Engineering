# Program 03 — Multiple Conditions

## 1. Objective

Understand how the logical AND operator (`&&`) translates into multiple conditional checks and short-circuit control flow.

## 2. Original C

```
int main(void)
{
    int a = 10;
    int b = 20;
    int c = 1;

    if (a < b && c != 0)
        return 1;

    return 0;
}
```

## 3. Behavioral Prediction

- **Expected output:** None.
    
- **Expected return value:** `1`.
    
- **Variables:** `a = 10`, `b = 20`, `c = 1`.
    
- **Control flow:** Both conditions are true, so return `1`.
    

## 4. Compilation

- **Compiler/version:** GCC 15.2.0 (MSYS2).
    
- **Architecture:** x86-64 Windows.
    
- **Build configuration:** `-O0 -g`.
    
- **Compilation command:**
    

```
gcc -O0 -g -o 03_multi_cond.exe 03_multi_cond.c
```

## 5. Assembly Observations

|Instruction|Effect|Interpretation|
|---|---|---|
|`cmp eax, [rbp-8]`|Compares `a` with `b`|Checks `a < b`.|
|`jge main+0x37`|Jumps if `a >= b` (signed)|Skips the second condition if the first is false.|
|`cmp [rbp-0Ch], 0`|Compares `c` with zero|Checks whether `c` is zero.|
|`je main+0x37`|Jumps if `c == 0`|Skips the success branch if the second condition is false.|
|`mov eax, 1`|Sets `EAX = 1`|Both conditions passed.|
|`jmp main+0x3c`|Jumps past the failure return|Avoids falling through into the other result.|
|`mov eax, 0`|Sets `EAX = 0`|At least one condition failed.|
|`ret`|Returns to the caller|Returns the value in `EAX`.|

**Key observation:** The first condition is checked before the second. If `a < b` is false, `jge` skips the check of `c`. This is **short-circuit evaluation**.

## 6. C Reconstruction

**My reconstruction:**

```
int main()
{
    int a = 10;
    int b = 20;
    int c = 1;

    if(a < b)
    {
        if(c != 0)
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
    
- **Difference:** I used nested `if` statements instead of `&&`.
    
- **Equivalence:** Both require `a < b` and `c != 0` to return `1`; otherwise, they return `0`.
    

## 7. Verification

- **Does it compile?** Yes
    
- **Expected return value:** `1`.
    
- **Test cases:**
    

|`a`|`b`|`c`|Expected return|
|---|---|---|---|
|10|20|1|1|
|20|10|1|0|
|10|20|0|0|

## 8. What I Misunderstood

No reconstruction mistake identified. Nested `if` statements can express the same logical requirement as `&&` in this example.

## 9. Key RE Lesson

The `&&` operator requires both conditions to be true. Compilers often implement this with conditional jumps that skip later checks when an earlier condition fails. This is short-circuit evaluation, and nested `if` statements can represent equivalent control flow.

## 10. Checkpoint

- Identify the two comparisons and conditional jumps.
    
- Explain how the branches implement logical AND.
    
- Reconstruct the program's logic.
    
- Explain why my reconstruction is equivalent.
    
- Compile and verify the program.
    
- Test what happens when the first condition is false and determine whether the second check is skipped.