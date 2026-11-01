# Program 01 — Simple If

## 1. Objective

Understand how a C `if` statement translates into a comparison (`cmp`) and conditional jump in x86-64 assembly.

## 2. Original C

```
int main(void)
{
    int a = 10;
    int result = 0;

    if (a > 5)
        result = 1;

    return result;
}
```

## 3. Behavioral Prediction

- **Expected output:** None.
    
- **Expected return value:** `1`.
    
- **Important variables:** `a = 10`, `result = 0 → 1`.
    
- **Expected control flow:** If `a <= 5`, skip the assignment. Otherwise, set `result = 1`.
    

## 4. Compilation

- **Compiler/version:** GCC 15.2.0 (MSYS2).
    
- **Architecture:** x86-64 Windows.
    
- **Build configuration:** `-O0 -g`.
    
- **Compilation command:**
    

```
gcc -O0 -g -o 01-sim_if.exe 01-Simple_if.c
```

## 5. Assembly Observations

|Instruction|Effect|Interpretation|
|---|---|---|
|`mov [rbp-8], 0Ah`|Stores `10` on the stack|Initializes `a`.|
|`mov [rbp-4], 0`|Stores `0` on the stack|Initializes `result`.|
|`cmp [rbp-8], 5`|Compares `a` with `5`|Updates CPU flags based on `a - 5`.|
|`jle main+0x28`|Jumps if `a <= 5` (signed)|Skips the assignment when `a > 5` is false.|
|`mov [rbp-4], 1`|Stores `1`|Executes the assignment when `a > 5`.|
|`mov eax, [rbp-4]`|Loads `result` into `EAX`|Prepares the return value.|
|`add rsp, 30h`|Restores the stack pointer|Cleans up the stack frame.|
|`pop rbp`|Restores the previous `RBP`|Completes the function epilogue.|
|`ret`|Returns to the caller|Ends the function.|

**Key observations:**

- `cmp` updates CPU flags; it does not store the subtraction result.
    
- `jle` means _jump if less than or equal_ using a signed comparison.
    
- When `a = 10`, `jle` is not taken, so `result` becomes `1`.
    
- `EAX` carries the integer return value.
    
- The prologue, stack allocation, and `__main` call are compiler/runtime setup rather than the core `if` logic.
    

## 6. C Reconstruction

**My reconstruction:**

```
int main()
{
    int a = 10;
    int b = 0;

    if(a > 5)
    {
        return 1;
    }

    return 0;
}
```

### Evaluation

- **Logic:** Correct.
    
- **Return value:** `1`, matching the original program.
    
- **Variable difference:** I used `b` instead of `result`. Since `b` is unused, it does not affect the behavior.
    
- **Control-flow difference:** My reconstruction returns directly inside the `if`, whereas the original assigns `1` to `result` and returns it afterward.
    
- **Equivalence:** Both programs return `1` when `a > 5` and `0` otherwise.
    

**Conclusion:** My reconstruction is behaviorally equivalent to the original for all possible values of `a`, assuming ordinary execution and no other observable side effects. The internal operations differ, but the return behavior is preserved.

## 7. Verification

- **Does it compile?** Yes
    
- **Expected return value:** `1` for the original input.
    
- **Test cases:**
    

|Value of `a`|Condition `a > 5`|Expected return|
|---|---|---|
|`10`|True|`1`|
|`5`|False|`0`|
|`4`|False|`0`|

Run the compiled program in PowerShell:

```
.\01_sim_if.exe
$LASTEXITCODE
```

The expected exit code is `1` for the original input.

## 8. What I Misunderstood

- I returned directly inside the `if` rather than assigning a value to a variable.
    
- These changes alter the implementation, but not the program's observable return behavior.

## 9. Key RE Lesson

A C `if` statement can compile into a comparison followed by a conditional jump. Here, `cmp` compares `a` with `5`, and `jle` skips the assignment when `a <= 5`. When the condition is true, execution continues and the program produces a return value of `1`. Reverse engineering often allows multiple valid C reconstructions because different source code can produce equivalent observable behavior.

## 10. Checkpoint

- Identify `cmp` and `jle`.
    
- Explain how the conditional jump implements the `if`.
    
- Reconstruct the program's behavior.
    
- Explain why my reconstruction is behaviorally equivalent.
    
- Compile and verify the program.
    
- Independently trace a modified version with different values of `a`.