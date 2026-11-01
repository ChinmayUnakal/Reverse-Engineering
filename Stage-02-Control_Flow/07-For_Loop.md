# Program 07 — For Loop

## 1. Objective

Understand how a C `for` loop is translated into x86-64 assembly. Learn to identify loop initialization, condition checking, increment operations, backward jumps, and repeated execution.

## 2. Original C

```
int main(void)
{
    int sum = 0;

    for (int i = 1; i <= 5; i++)
        sum += i;

    return sum;
}
```

## 3. Behavioral Prediction

- **Expected output:** None (no printing).
    
- **Expected return value:** 15.
    
- **Important variable values:**
    
    - `sum = 0` initially.
        
    - `i = 1` initially.
        
    - Final `sum = 15`.
        
    - Final `i = 6`.
        
- **Expected branches or function calls:**
    
    - Initialize variables.
        
    - Compare `i` with 5.
        
    - Execute the loop body if `i <= 5`.
        
    - Increment `i`.
        
    - Jump back to the condition check.
        
    - Exit when the condition becomes false.
        

## 4. Compilation

- **Compiler/version:** GCC 15.2.0 (MSYS2).
    
- **Architecture:** x86-64 Windows.
    
- **Build configuration:** `-O0 -g`.
    
- **Compilation command:**

```
gcc -O0 -g -o 07-for.exe 07-for_loop.c
```

## 5. Assembly Observations

|Instruction|Register or memory effect|Interpretation|
|---|---|---|
|`mov [rbp-4], 0`|Stores 0|Initializes `sum`.|
|`mov [rbp-8], 1`|Stores 1|Initializes loop variable `i`.|
|`jmp main+0x27`|Jumps to condition check|Checks the condition before executing the body.|
|`mov eax, [rbp-8]`|Loads `i` into EAX|Retrieves the current loop variable.|
|`add [rbp-4], eax`|Adds `i` to `sum`|Executes the loop body.|
|`add [rbp-8], 1`|Increments `i`|Performs `i++`.|
|`cmp [rbp-8], 5`|Compares `i` with 5|Checks the loop condition.|
|`jle main+0x1d`|Jumps backward if `i <= 5`|Repeats the loop.|
|`mov eax, [rbp-4]`|Loads `sum` into EAX|Prepares the return value.|
|`ret`|Returns from function|Returns 15.|

### Important Assembly Instructions

**1.** **jmp** **— Unconditional Jump**

```
jmp main+0x27
```

Initially jumps directly to the loop condition check, skipping the loop body.

**2.** **cmp** **— Comparison**

```
cmp dword ptr [rbp-8], 5
```

Compares the current value of `i` against 5 and updates CPU flags.

**3.** **jle** **— Jump if Less or Equal**

```
jle main+0x1d
```

If `i <= 5`, execution jumps backward to the loop body. Otherwise, execution continues after the loop.

**4.** **add** **— Addition and Increment**

```
add dword ptr [rbp-4], eax
add dword ptr [rbp-8], 1
```

- First instruction: Adds the current `i` to `sum`.
    
- Second instruction: Increments `i` by 1.

### Loop Execution

|Iteration|`i`|`sum`|
|---|---|---|
|1|1|1|
|2|2|3|
|3|3|6|
|4|4|10|
|5|5|15|
|Exit|6|15|

## 6. C Reconstruction

**My reconstruction:**

```
int main()
{
    int a = 0;
    int b = 1;

    for(int b; b <= 5; b++)
    {
        a += b;
    }

    return a;
}
```

**Issue discovered:**

- `int b` inside the `for` loop declares a new variable.
    
- This new variable shadows the previously initialized `b = 1`.
    
- The new variable is uninitialized.
    
- Reading it in `b <= 5` causes undefined behavior.
    

**Minimal correction (preserving my reconstruction):**

```
int main()
{
    int a = 0;
    int b = 1;

    for(; b <= 5; b++)
    {
        a += b;
    }

    return a;
}
```

The corrected reconstruction preserves the original program's behavior.

## 7. Verification

- **Does it compile?** The corrected reconstruction compiles.
    
- **Does it behave like the original?** Yes, after correcting the uninitialized variable.
    
- **Test cases:**
    
    - `b = 1`, condition `b <= 5` → returns 15.
        
    - Loop from 1 to 3 → returns 6.
        
    - Loop from 1 to 1 → returns 1.
        
    - Condition initially false → returns 0.
        
- **Differences discovered:**
    
    - Original variables: `sum` and `i`.
        
    - Reconstructed variables: `a` and `b`.
        
    - Variable names differ, but the corrected logic is equivalent.
        
    - The original declares its loop variable inside the `for` statement, while my reconstruction declares it outside.
        

## 8. What I Misunderstood

- Declared `b` again inside the `for` loop instead of using the previously initialized variable.
    
- Accidentally created a new uninitialized variable through variable shadowing.
    
- Initially, the purpose of the first `jmp` was unclear. It jumps directly to the condition check before the first iteration.
    

## 9. Key RE Lesson

A `for` loop is implemented through initialization, condition checking, loop body execution, increment, and backward conditional jumps.

The `cmp` and `jle` instructions implement the loop condition, while `jmp` directs execution to the initial condition check.

Recognizing backward jumps and their associated comparisons helps identify loops when reconstructing C code from assembly.

## 10. Checkpoint

- Identified loop initialization.
    
- Identified the loop body.
    
- Identified the increment operation.
    
- Identified `cmp` and `jle`.
    
- Understood the initial unconditional jump.
    
- Reconstruct a modified loop independently.