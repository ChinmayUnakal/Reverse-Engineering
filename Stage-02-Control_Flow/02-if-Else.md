# Program 02 — If-Else

## 1. Objective

Understand how an `if-else` statement becomes a conditional branch, alternative return values, and an unconditional jump in x86-64 assembly.

## 2. Original C

```
int main(void)
{
    int a = 7;
    int b = 10;

    if (a > b)
        return 1;
    else
        return 2;
}
```

## 3. Behavioral Prediction

- **Expected output:** None.
    
- **Expected return value:** `2`.
    
- **Variables:** `a = 7`, `b = 10`.
    
- **Control flow:** Since `7 > 10` is false, execute the `else` branch.
    

## 4. Compilation

- **Compiler/version:** GCC 15.2.0 (MSYS2).
    
- **Architecture:** x86-64 Windows.
    
- **Build configuration:** `-O0 -g`.
    
- **Compilation command:**
```
gcc -O0 -g -o 02_if_else.exe 02_if_else.c
```

## 5. Assembly Observations

|Instruction|Effect|Interpretation|
|---|---|---|
|`mov [rbp-4], 7`|Stores `7`|Initializes `a`.|
|`mov [rbp-8], 0Ah`|Stores `10`|Initializes `b`.|
|`mov eax, [rbp-4]`|Loads `a` into `EAX`|Prepares the first operand.|
|`cmp eax, [rbp-8]`|Compares `a` with `b`|Updates CPU flags based on `a - b`.|
|`jle main+0x2a`|Jumps if `a <= b` (signed)|Selects the `else` branch when the condition is false.|
|`mov eax, 1`|Sets `EAX = 1`|Return value for the `if` branch.|
|`jmp main+0x2f`|Jumps past the `else` return|Prevents execution from falling through into the `else` branch.|
|`mov eax, 2`|Sets `EAX = 2`|Return value for the `else` branch.|
|`ret`|Returns to the caller|Returns the value in `EAX`.|

**Key observation:** `jle` is taken because `7 <= 10`. Execution skips `mov eax, 1` and reaches `mov eax, 2`. Therefore, the return value is `2`.

## 6. C Reconstruction

**My reconstruction:**

```
int main()
{
    int a = 7;
    int b = 10;

    if(a > b)
    {
        return 1;
    }

    return 2;
}
```

### Evaluation

- **Logic:** Correct.
    
- **Expected return value:** `2`.
    
- **Differences:** My reconstruction omits the explicit `else` keyword.
    
- **Equivalence:** Both programs return `1` when `a > b` and `2` otherwise.
    

## 7. Verification

- **Does it compile?** Yes
    
- **Expected return value:** `2`.
    
- **Test cases:**
    

|`a`|`b`|Expected return|
|---|---|---|
|7|10|2|
|12|10|1|
|10|10|2|

Run in PowerShell:

```
.\02_if_else.exe
$LASTEXITCODE
```

## 8. What I Misunderstood

No reconstruction mistake identified.

## 9. Key RE Lesson

An `if-else` statement can compile into a conditional jump that selects one of two execution paths. Here, `jle` branches to the `else` path when `a <= b`. The unconditional `jmp` prevents the `if` branch from falling through into the `else` branch. Both paths place their return value in `EAX`.

## 10. Checkpoint

- Identify `cmp` and `jle`.
    
- Explain how the conditional jump selects a branch.
    
- Reconstruct the program's logic.
    
- Explain why my reconstruction is equivalent.
    
- Compile and verify the program.
    
- Independently trace modified values of `a` and `b`.