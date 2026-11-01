# Program 08 — While Loop

## 1. Objective

Understand how a C `while` loop is translated into x86-64 assembly. Learn to identify loop initialization, condition checking, loop body execution, backward jumps, and arithmetic operations implemented using bitwise instructions.

## 2. Original C

```
int main(void)
{
    int i = 5;
    int result = 1;

    while (i > 0)
    {
        result *= 2;
        i--;
    }

    return result;
}
```

## 3. Behavioral Prediction

- **Expected output:** None (no printing).
    
- **Expected return value:** 32.
    
- **Important variable values:**
    
    - `i = 5` initially.
        
    - `result = 1` initially.
        
    - `result` doubles during every iteration.
        
    - Final `i = 0`.
        
    - Final `result = 32`.
        
- **Expected branches or function calls:**
    
    - Initialize `i` and `result`.
        
    - Check whether `i > 0`.
        
    - Multiply `result` by 2.
        
    - Decrement `i`.
        
    - Jump back to the condition check.
        
    - Exit when `i` becomes 0.
        

## 4. Compilation

- **Compiler/version:** GCC 15.2.0 (MSYS2).
    
- **Architecture:** x86-64 Windows.
    
- **Build configuration:** `-O0 -g`.
    
- **Compilation command:**
    

```
gcc -O0 -g -o 08-while.exe 08-while_loop.c
```

## 5. Assembly Observations

|Instruction|Register or memory effect|Interpretation|
|---|---|---|
|`mov [rbp-4], 5`|Stores 5|Initializes `i`.|
|`mov [rbp-8], 1`|Stores 1|Initializes `result`.|
|`jmp main+0x24`|Jumps to condition check|Checks the condition before the first iteration.|
|`shl [rbp-8], 1`|Shifts `result` left by one bit|Multiplies `result` by 2.|
|`sub [rbp-4], 1`|Decrements `i`|Performs `i--`.|
|`cmp [rbp-4], 0`|Compares `i` with 0|Checks the loop condition.|
|`jg main+0x1d`|Jumps backward if `i > 0`|Repeats the loop.|
|`mov eax, [rbp-8]`|Loads `result` into EAX|Prepares the return value.|
|`ret`|Returns from function|Returns 32.|

### Important Assembly Instructions

**1.** **jmp** **— Unconditional Jump**

```
jmp main+0x24
```

Initially jumps directly to the condition check instead of executing the loop body immediately.

**2.** **shl** **— Shift Left**

```
shl dword ptr [rbp-8], 1
```

Shifts the bits of `result` one position to the left.

For positive integers, shifting left by one bit is equivalent to multiplying by 2, provided the result remains representable.

Example:

```
1  = 00000001
2  = 00000010
4  = 00000100
8  = 00001000
16 = 00010000
32 = 00100000
```

Therefore:

```
result *= 2;
```

was implemented using:

```
shl dword ptr [rbp-8], 1
```

**3.** **sub** **— Subtraction**

```
sub dword ptr [rbp-4], 1
```

Subtracts 1 from `i`, implementing:

```
i--;
```

**4.** **cmp** **— Comparison**

```
cmp dword ptr [rbp-4], 0
```

Compares `i` against zero and updates CPU flags.

**5.** **jg** **— Jump if Greater**

```
jg main+0x1d
```

If the signed comparison indicates `i > 0`, execution jumps backward to the loop body.

Otherwise, execution continues after the loop.

### Loop Execution

|Iteration|`i`|`result`|
|---|---|---|
|Initial|5|1|
|1|4|2|
|2|3|4|
|3|2|8|
|4|1|16|
|5|0|32|
|Exit|0|32|

## 6. C Reconstruction

**My reconstruction:**

```
int main()
{
    int a = 5;
    int b = 1;

    while(a > 0)
    {
        b = b * 2;
        a--;
    }

    return b;
}
```

My reconstruction uses different variable names and an explicit multiplication expression, but preserves the original program's logic.

- Original `i` corresponds to reconstructed `a`.
    
- Original `result` corresponds to reconstructed `b`.
    
- Original `result *= 2` corresponds to `b = b * 2`.
    
- Both programs execute five iterations and return 32.
    

## 7. Verification

- **Does it compile?** Yes.
    
- **Does it behave like the original?** Yes.
    
- **Test cases:**
    
    - `a = 5`, `b = 1` → returns 32.
        
    - `a = 3`, `b = 1` → returns 8.
        
    - `a = 1`, `b = 1` → returns 2.
        
    - `a = 0`, `b = 1` → returns 1.
        
- **Differences discovered:**
    
    - Variable names differ (`i`/`result` versus `a`/`b`).
        
    - The original uses `result *= 2`, while the reconstruction uses `b = b * 2`.
        
    - GCC implements multiplication by 2 using `shl` in the observed assembly.
        
    - The loop behavior and return value are equivalent.
        

## 8. What I Misunderstood

- The `shl` instruction initially might look like a bitwise operation unrelated to multiplication.
    
- However, shifting a positive integer left by one bit multiplies it by 2, provided there is no signed overflow.
    
- The initial `jmp` skips the loop body and directs execution to the condition check.
    
- The `jg` instruction controls whether another iteration takes place.

## 9. Key RE Lesson

A `while` loop typically uses an initial jump to reach its condition check, followed by a conditional backward jump to repeat the loop body.

The `cmp` and `jg` instructions implement the loop condition, while `sub` performs the decrement operation.

Compilers can also replace arithmetic operations with equivalent instructions. For example, multiplying a positive integer by 2 can be implemented using `shl`. Recognizing these instruction substitutions is important when reconstructing C from assembly.

## 10. Checkpoint

- Identified variable initialization.
    
- Identified the loop body.
    
- Identified the decrement operation.
    
- Identified `cmp` and `jg`.
    
- Understood the initial unconditional jump.
    
- Identified `shl` as multiplication by 2 in this context.