# Program 09 — Do-While Loop

## 1. Objective

Understand how a C `do-while` loop translates into x86-64 assembly. Learn to identify post-condition checking, loop body execution, and conditional backward jumps. Understand how assembly reveals that a loop executes at least once.

## 2. Original C

```
int main(void)
{
    int i = 0;

    do
    {
        i++;
    } while (i < 3);

    return i;
}
```

## 3. Behavioral Prediction

- **Expected output:** None (no printing).
    
- **Expected return value:** 3.
    
- **Important variable values:**
    
    - `i = 0` initially.
        
    - Iteration 1: `i = 1`.
        
    - Iteration 2: `i = 2`.
        
    - Iteration 3: `i = 3`.
        
    - Final `i = 3`.
        
- **Expected branches or function calls:**
    
    - Initialize `i`.
        
    - Execute the loop body.
        
    - Increment `i`.
        
    - Check whether `i < 3`.
        
    - Jump backward if the condition is true.
        
    - Exit when the condition becomes false.
        

## 4. Compilation

- **Compiler/version:** GCC 15.2.0 (MSYS2).
    
- **Architecture:** x86-64 Windows.
    
- **Build configuration:** `-O0 -g`.
    
- **Compilation command:**

```
gcc -O0 -g -o 09-do-while.exe 09-do-while.c
```

## 5. Assembly Observations

|Instruction|Register or memory effect|Interpretation|
|---|---|---|
|`mov [rbp-4], 0`|Stores 0|Initializes `i`.|
|`add [rbp-4], 1`|Increments `i`|Executes `i++`.|
|`cmp [rbp-4], 2`|Compares `i` with 2|Checks whether `i <= 2`.|
|`jle main+0x14`|Jumps backward if `i <= 2`|Repeats the loop body.|
|`mov eax, [rbp-4]`|Loads `i` into EAX|Prepares the return value.|
|`ret`|Returns from function|Returns 3.|

### Important Assembly Instructions

**1.** **add** **— Increment**

```
add dword ptr [rbp-4], 1
```

Adds 1 to `i`, implementing:

```
i++;
```

**2.** **cmp** **— Comparison**

```
cmp dword ptr [rbp-4], 2
```

Compares `i` against 2.

The original condition is `i < 3`, which for integers is equivalent to `i <= 2`.

**3.** **jle** **— Jump if Less or Equal**

```
jle main+0x14
```

If `i <= 2`, execution jumps backward to the increment instruction.

Otherwise, execution continues after the loop.

### Loop Execution

|Iteration|`i` before increment|`i` after increment|Condition|
|---|---|---|---|
|1|0|1|True|
|2|1|2|True|
|3|2|3|False|

Final return value: **3**.

## 6. C Reconstruction

**My reconstruction:**

```
int main()
{
    int a = 0;

    do{
        a++;
    }while(a <= 2);

    return a;
}
```

My reconstruction is logically equivalent to the original.

- Original `i` corresponds to reconstructed `a`.
    
- Original condition `i < 3` corresponds to `a <= 2`.
    
- Both increment the variable before checking the condition.
    
- Both execute three iterations and return 3.
    

## 7. Verification

- **Does it compile?** Yes.
    
- **Does it behave like the original?** Yes.
    
- **Test cases:**
    
    - Initial value 0, condition `a <= 2` → returns 3.
        
    - Initial value 5, condition `a <= 2` → returns 6.
        
    - Initial value 2, condition `a <= 2` → returns 3.
        
- **Differences discovered:**
    
    - Variable names differ (`i` versus `a`).
        
    - Condition syntax differs (`i < 3` versus `a <= 2`).
        
    - Both conditions are equivalent for integer values.
        

## 8. Some things to look at

- The assembly compares `i` against 2 instead of 3 because the compiler transformed `i < 3` into the equivalent integer comparison `i <= 2`.
    
- Unlike the previous `for` and `while` examples, there is no initial unconditional jump to the condition check.
    
- The condition is checked after executing the loop body.

## 9. Key RE Lesson

A `do-while` loop executes its body before checking its condition, guaranteeing at least one iteration.

In assembly, this can be recognized by a loop body followed by a comparison and a conditional backward jump. Unlike a typical `while` loop, there is no need to jump over the body to perform an initial condition check.

## 10. Checkpoint

- Identified variable initialization.
    
- Identified the loop body.
    
- Identified the increment operation.
    
- Identified `cmp` and `jle`.
    
- Understood the backward conditional jump.
    
- Reconstructed the original loop.