# Program 10 — Nested Loop

## 1. Objective

Understand how nested loops translate into x86-64 assembly. Learn to identify multiple loop conditions, separate inner and outer loop control flow, and track how variables change across iterations.

## 2. Original C

```
int main(void)
{
    int count = 0;

    for (int i = 0; i < 3; i++)
    {
        for (int j = 0; j < 2; j++)
        {
            count++;
        }
    }

    return count;
}
```

## 3. Behavioral Prediction

- **Expected output:** None (no printing).
    
- **Expected return value:** 6.
    
- **Important variable values:**
    
    - `count = 0` initially.
        
    - Outer loop executes 3 times.
        
    - Inner loop executes 2 times per outer iteration.
        
    - Total iterations = 3 × 2 = 6.
        
    - Final `count = 6`.
        
- **Expected branches or function calls:**
    
    - Initialize `count` and outer loop variable `i`.
        
    - Check the outer loop condition.
        
    - Initialize inner loop variable `j`.
        
    - Check the inner loop condition.
        
    - Increment `count` and `j`.
        
    - Repeat the inner loop until its condition becomes false.
        
    - Increment `i` and repeat the outer loop.
        
    - Exit when the outer condition becomes false.
        

## 4. Compilation

- **Compiler/version:** GCC 15.2.0 (MSYS2).
    
- **Architecture:** x86-64 Windows.
    
- **Build configuration:** `-O0 -g`.
    
- **Compilation command:**

```
gcc -O0 -g -o 10-nested-loop.exe 10-nested_loop.c
```

## 5. Assembly Observations

|Instruction|Register or memory effect|Interpretation|
|---|---|---|
|`mov [rbp-4], 0`|Stores 0|Initializes `count`.|
|`mov [rbp-8], 0`|Stores 0|Initializes outer loop variable `i`.|
|`jmp main+0x38`|Jumps to outer condition|Checks outer loop before execution.|
|`mov [rbp-0Ch], 0`|Stores 0|Initializes inner loop variable `j`.|
|`jmp main+0x2e`|Jumps to inner condition|Checks inner loop before execution.|
|`add [rbp-4], 1`|Increments `count`|Executes `count++`.|
|`add [rbp-0Ch], 1`|Increments `j`|Advances inner loop.|
|`cmp [rbp-0Ch], 1`|Compares `j` with 1|Checks whether `j <= 1`.|
|`jle main+0x26`|Jumps backward if `j <= 1`|Repeats inner loop.|
|`add [rbp-8], 1`|Increments `i`|Advances outer loop.|
|`cmp [rbp-8], 2`|Compares `i` with 2|Checks whether `i <= 2`.|
|`jle main+0x1d`|Jumps backward if `i <= 2`|Repeats outer loop.|
|`mov eax, [rbp-4]`|Loads `count` into EAX|Prepares return value.|
|`ret`|Returns from function|Returns 6.|

### Important Assembly Instructions

**1.** `**cmp**` **— Two Separate Loop Conditions**

Inner loop:

```
cmp dword ptr [rbp-0Ch], 1
```

Checks whether `j <= 1`, equivalent to the original `j < 2` for integers.

Outer loop:

```
cmp dword ptr [rbp-8], 2
```

Checks whether `i <= 2`, equivalent to the original `i < 3`.

**2.** `**jle**` **— Conditional Backward Jumps**

Inner loop:

```
jle main+0x26
```

Repeats the inner loop if `j <= 1`.

Outer loop:

```
jle main+0x1d
```

Repeats the outer loop if `i <= 2`.

Notice that both loops use `jle`, but jump to different addresses.

**3.** `**add**` **— Loop Variable and Counter Updates**

```
add dword ptr [rbp-4], 1
add dword ptr [rbp-0Ch], 1
add dword ptr [rbp-8], 1
```

These instructions:

- Increment `count`.
    
- Increment inner loop variable `j`.
    
- Increment outer loop variable `i`.
    

### Understanding Nested Loop Execution

|Outer iteration (`i`)|Inner iterations (`j`)|Count|
|---|---|---|
|0|0, 1|2|
|1|0, 1|4|
|2|0, 1|6|

Final return value: **6**.

## 6. C Reconstruction

**My reconstruction:**

```
int main()
{
    int a = 0;
    int b = 0;

    while(b < 3)
    {
        for(int i = 0; i < 2; i++)
        {
            a++;
        }

        b++;
    }

    return a;
}
```

My reconstruction is logically equivalent to the original.

- Original `count` corresponds to reconstructed `a`.
    
- Original outer loop variable `i` corresponds to reconstructed `b`.
    
- Original inner loop variable `j` corresponds to reconstructed `i`.
    
- Both execute the inner loop twice for each of the three outer iterations.
    
- Both increment the counter six times and return 6.
    

## 7. Verification

- **Does it compile?** Yes.
    
- **Does it behave like the original?** Yes.
    
- **Test cases:**
    
    - Outer loop: 3 iterations, inner loop: 2 iterations → returns 6.
        
    - Outer loop: 2 iterations, inner loop: 2 iterations → returns 4.
        
    - Outer loop: 3 iterations, inner loop: 1 iteration → returns 3.
        
    - Outer loop initially false → returns 0.
        
- **Differences discovered:**
    
    - Original uses two `for` loops.
        
    - Reconstruction uses an outer `while` loop and inner `for` loop.
        
    - Variable names differ.
        
    - Loop structures differ, but the execution behavior is equivalent.
        

## 8. What I Misunderstood

- Initially, identifying which conditional jump belonged to which loop could be confusing.
    
- The inner loop has its own initialization, condition check, increment, and backward jump.
    
- The outer loop has a separate condition check and backward jump.
    
- The inner loop is reinitialized every time the outer loop starts another iteration.
    

## 9. Key RE Lesson

Nested loops produce multiple independent loop-control structures in assembly. Each loop has its own initialization, comparison, and conditional backward jump.

To reconstruct nested loops, identify the jump targets and determine which instructions belong to the inner and outer loops. Also track variable initialization carefully, especially when an inner loop is reinitialized during every outer iteration.

Different C loop constructs can implement equivalent nested-loop behavior.

## 10. Checkpoint

- Identified counter initialization.
    
- Identified outer loop initialization.
    
- Identified inner loop initialization.
    
- Identified both comparisons.
    
- Identified both conditional backward jumps.
    
- Distinguished inner and outer loop control flow.
    
- Reconstructed the nested loop.