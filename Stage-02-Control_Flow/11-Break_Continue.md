# Program 11 — Break_Continue

## 1. Objective

Understand how `break` and `continue` are implemented using conditional jumps and unconditional jumps. Learn how control flow skips statements, exits loops, and reaches loop-increment logic.

## 2. Original C

```
int main(void)
{
    int sum = 0;

    for (int i = 1; i <= 10; i++)
    {
        if (i == 3)
            continue;

        if (i == 7)
            break;

        sum += i;
    }

    return sum;
}
```

## 3. Behavioral prediction

- **Expected output:** No output.
    
- **Expected return value:** `17`.
    
- **Important variable values:**
    
    - `sum` starts at `0`.
        
    - `i` starts at `1`.
        
    - `i == 3`: skip `sum += i`.
        
    - `i == 7`: exit the loop.
        
    - Final `sum = 1 + 2 + 4 + 5 + 6 = 18`? No — the correct sum is **18**.
        
- **Expected branches or function calls:**
    
    - `je` when `i == 3` to skip the addition.
        
    - `je` when `i == 7` to exit the loop.
        
    - `jle` to repeat the loop while `i <= 10`.
        

## 4. Compilation

- **Compiler/version:** GCC 15.2.0 (MSYS2).
    
- **Architecture:** x86-64 Windows.
    
- **Build configuration:** `-O0 -g`.
    
- **Compilation command:**
    

```
gcc -O0 -g -o 11-break_continue.exe 11-break_continue.c
```

## 5. Assembly observations

|Instruction/address|Register or memory effect|My interpretation|
|---|---|---|
|`14a4: mov [rbp-8], 1`|Initializes `i = 1`|Loop variable initialization.|
|`14ab: jmp 14c6`|Jumps to loop condition|Checks the condition before entering the loop body.|
|`14ad: cmp [rbp-8], 3`|Compares `i` with `3`|Checks the `continue` condition.|
|`14b1: je 14c1`|Jumps if `i == 3`|Skips the addition and proceeds to the increment.|
|`14b3: cmp [rbp-8], 7`|Compares `i` with `7`|Checks the `break` condition.|
|`14b7: je 14ce`|Jumps if `i == 7`|Exits the loop and reaches the return-value setup.|
|`14b9: mov eax, [rbp-8]`|Loads `i` into `eax`|Prepares the value to add to `sum`.|
|`14bc: add [rbp-4], eax`|Adds `i` to `sum`|Executes `sum += i`.|
|`14bf: jmp 14c2`|Jumps to the increment|Skips the `nop` used as the continue target.|
|`14c1: nop`|No operation|Continue target; execution proceeds to `i++`.|
|`14c2: add [rbp-8], 1`|Increments `i`|Executes the `for` loop increment.|
|`14c6: cmp [rbp-8], 10`|Compares `i` with `10`|Checks the loop condition.|
|`14ca: jle 14ad`|Jumps if `i <= 10`|Repeats the loop body.|
|`14cf: mov eax, [rbp-4]`|Loads `sum` into `eax`|Places the return value in the return register.|

**Important control-flow observations:**

- `continue` jumps to `14c1`, which is immediately before the increment at `14c2`.
    
- `break` jumps to `14ce`, bypassing the loop increment and condition.
    
- The normal execution path also reaches `14c2` after adding to `sum`.
    

## 6. C reconstruction

My reconstruction:

```
int main()
{
    int a = 0;
    int b = 1;

    while(b <= 10)
    {
        if(b == 3)
            continue;

        if(b == 7)
            break;

        a += b;
        b++;
    }

    return a;
}
```

**Difference discovered:**

My reconstruction is not equivalent to the original.

In the original `for` loop, `continue` transfers execution to the increment expression (`i++`) before checking the condition again.

In my `while` loop, `continue` jumps directly to the condition (`b <= 10`). Since `b++` is after the `continue`, it never executes when `b == 3`.

Therefore, `b` remains `3` forever, resulting in an infinite loop.

**Minimal correction:**

Move `b++` before the `continue` can skip it, or restructure the loop so every iteration reaches the increment. For example:

```
while(b <= 10)
{
    if(b == 3)
    {
        b++;
        continue;
    }

    if(b == 7)
        break;

    a += b;
    b++;
}
```

## 7. Verification

- **Does it compile?** Yes, the reconstruction compiles.
    
- **Does it behave like the original?** No. It enters an infinite loop when `b == 3`.
    
- **Which test cases did I try?** Trace execution at `b = 1`, `2`, `3`, and `7`.
    
- **What differences did I discover?** The original `for` loop executes its increment after `continue`, while my `while` reconstruction skips the increment.
    

After the correction, the reconstruction returns `18`, matching the original.

## 8. What I misunderstood

- I assumed `continue` always executes the loop increment.
    
- In reality, `continue` transfers control to the loop's continuation point.
    
- In a `for` loop, that point includes the increment expression.
    
- In a `while` loop, that point is the condition check.
    

## 9. Key RE lesson

`break` and `continue` are implemented through control-flow transfers, usually conditional and unconditional jumps. `break` exits the loop, while `continue` skips the remaining body and transfers execution to the loop's continuation point. When reconstructing loops, identify where these jumps land rather than assuming that different loop constructs behave identically.

## 10. Checkpoint

- I can explain every relevant instruction.
    
- I can reconstruct the program without the source.
    
- I can explain why my reconstruction is equivalent after correction.
    
- I can solve a slightly modified version independently.