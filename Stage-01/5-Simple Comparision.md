# Program 05 — Simple Comparison

## 1. Objective

This program teaches me how to reverse-engineer conditional logic from x86-64 assembly.

- Identify comparison instructions such as `CMP`.
    
- Understand conditional jumps such as `JGE`.
    
- Track how CPU flags control program execution.
    
- Reconstruct `if/else` logic from branch instructions.
    
- Understand how a function returns different values depending on a condition.
    

## 2. Original C

```
int main(void)
{
    int a = 10;
    int b = 20;

    if (a < b)
        return 1;
    else
        return 0;
}
```

## 3. Behavioral Prediction

- **Expected output:** No output; the program does not print anything.
    
- **Expected return value:** `1`, because `10 < 20` is true.
    
- **Important variable values:**
    
    - `a = 10` (`0Ah`)
        
    - `b = 20` (`14h`)
        
    - Comparison: `10 < 20` → true.
        
- **Expected branches or function calls:**
    
    - GCC runtime helper: `__main`.
        
    - `CMP` compares `a` and `b`.
        
    - `JGE` jumps when `a >= b`.
        
    - The program returns `1` when `a < b`, otherwise `0`.
        

## 4. Compilation

- **Compiler:** GCC 15.2.0 (MSYS2)
    
- **Architecture:** Windows x86-64.
    
- **Build configuration:** `-O0 -g`.
    
- **Compilation command:**
    

```
gcc -O0 -g -o sim_com.exe 05_simple_comparison.c
```

## 5. Assembly Observations

|Instruction/address|Register or memory effect|Interpretation|
|---|---|---|
|`19f6149d: mov [rbp-4], 0Ah`|Stores 10|Initializes `a`.|
|`19f614a4: mov [rbp-8], 14h`|Stores 20|Initializes `b`.|
|`19f614ab: mov eax, [rbp-4]`|`EAX = 10`|Loads `a`.|
|`19f614ae: cmp eax, [rbp-8]`|Compares 10 with 20|Sets CPU flags based on `a - b`, without saving the subtraction result.|
|`19f614b1: jge ...14ba`|Jumps if `a >= b`|False for 10 and 20, so execution falls through.|
|`19f614b3: mov eax, 1`|`EAX = 1`|Sets the return value for `a < b`.|
|`19f614b8: jmp ...14bf`|Jumps to the epilogue|Skips the `return 0` path.|
|`19f614ba: mov eax, 0`|`EAX = 0`|Sets the return value when `a >= b`.|
|`19f614bf: add rsp, 30h`|Restores stack pointer|Releases allocated stack space.|
|`19f614c3: pop rbp`|Restores `RBP`|Restores the previous frame pointer.|
|`19f614c4: ret`|Returns to caller|Returns the value in `EAX`.|

### Understanding `CMP` and `JGE`

```
mov eax, dword ptr [rbp-4]
cmp eax, dword ptr [rbp-8]
jge  sim_com!main+0x2a
```

`CMP` sets CPU flags as if it calculated:

```
EAX - [rbp-8]
10 - 20 = -10
```

It does not store `-10` in `EAX`. Instead, it updates flags used by conditional jumps.

`JGE` means **Jump if Greater or Equal**, using a signed comparison.

- If `a >= b`, jump to `mov eax, 0`.
    
- If `a < b`, do not jump; execution reaches `mov eax, 1`.
    

For this program, `10 < 20`, so the jump is not taken and the function returns `1`.

**Important RE insight:** The assembly uses `JGE`, which looks like the opposite of the original condition `a < b`. That is because the compiler has arranged the branch to jump to the `else` path when the original condition is false.

## 6. C Reconstruction

```
int main(void)
{
    int a = 10;
    int b = 20;

    if (a >= b)
        return 0;

    return 1;
}
```

This reconstruction is behaviorally equivalent to the original for all defined values of `a` and `b`: it returns `1` when `a < b` and `0` otherwise.

## 7. Verification

- **Does it compile?** It should compile successfully.
    
- **Does it behave like the original?** Yes, if the return value is `1` for `a = 10`, `b = 20`.
    
- **Test cases:**
    

|`a`|`b`|Condition `a < b`|Return value|
|---|---|---|---|
|10|20|True|1|
|20|10|False|0|
|10|10|False|0|

Compile and test:

```
gcc -O0 -g -o sim_com.exe 05_simple_comparison.c
.\sim_com.exe
$LASTEXITCODE
```

Expected exit status for the original values: `1`.

- **Differences discovered:** The reconstructed source expresses the logic using `a >= b` rather than the original `a < b`. Both forms are logically equivalent when the return paths are reversed accordingly.

## 8. Watch out for these :

- `CMP` does not store a subtraction result; it updates CPU flags.
    
- `JGE` checks whether a signed value is greater than or equal to another value.
    
- A conditional jump may test the opposite of the original C condition because it branches directly to the `else` path.
    
- The return value is placed in `EAX`; it is not automatically printed to the console.

## 9. Key RE Lesson

Conditional logic can be reconstructed by combining `CMP`, the flags it sets, and the conditional jump that follows. The jump may test the opposite of the condition written in C because the compiler chooses a convenient control-flow layout. I must trace both branch paths and identify which value reaches `EAX` before `RET`.

## 10. Checkpoint

- I can explain how `CMP` sets CPU flags.
    
- I can explain what `JGE` means for a signed comparison.
    
- I can trace both possible execution paths.
    
- I can reconstruct the equivalent `if/else` logic without the original source.
    
- I can explain why the program returns `1` without printing anything.
    
- I can solve a modified comparison independently.

