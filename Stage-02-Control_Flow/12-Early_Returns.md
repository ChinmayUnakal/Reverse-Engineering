# Program 12 — Early_Returns

## 1. Objective

Understand how early returns are implemented using conditional and unconditional jumps. Learn to identify multiple return paths and how a compiler directs each path toward a common function epilogue.

## 2. Original C

```
int main(void)
{
    int value = -5;

    if (value < 0)
        return 1;

    if (value == 0)
        return 2;

    return 3;
}
```

## 3. Behavioral prediction

- **Expected output:** No output.
    
- **Expected return value:** `1`.
    
- **Important variable values:**
    
    - `value = -5`.
        
    - `value < 0` evaluates to true.
        
    - The first return executes; the remaining conditions are skipped.
        
- **Expected branches or function calls:**
    
    - First condition takes the negative-value path.
        
    - Returns `1` without evaluating the second condition.
        

## 4. Compilation

- **Compiler/version:** GCC 15.2.0 (MSYS2).
    
- **Architecture:** x86-64 Windows.
    
- **Build configuration:** `-O0 -g`.
    
- **Compilation command:**
    

```
gcc -O0 -g -o 12-early_returns.exe 12-early_returns.c
```

## 5. Assembly observations

|Instruction/address|Register or memory effect|My interpretation|
|---|---|---|
|`149d: mov [rbp-4], 0FFFFFFFBh`|Stores `0xFFFFFFFB` in the local variable|Initializes `value` to `-5`.|
|`14a4: cmp [rbp-4], 0`|Compares `value` with zero|Checks whether the value is negative.|
|`14a8: jns 14b1`|Jumps if the sign flag is not set|Skips the first return when `value` is non-negative.|
|`14aa: mov eax, 1`|Places `1` in `eax`|Sets the first return value.|
|`14af: jmp 14c3`|Jumps to the function epilogue|Skips the remaining conditions and returns.|
|`14b1: cmp [rbp-4], 0`|Compares `value` with zero|Checks the second condition.|
|`14b5: jne 14be`|Jumps if the value is not zero|Skips the second return when `value != 0`.|
|`14b7: mov eax, 2`|Places `2` in `eax`|Sets the second return value.|
|`14bc: jmp 14c3`|Jumps to the function epilogue|Skips the final return.|
|`14be: mov eax, 3`|Places `3` in `eax`|Sets the default return value.|
|`14c3: add rsp, 30h`|Restores stack space|Function epilogue.|
|`14c7: pop rbp`|Restores the previous frame pointer|Completes stack-frame cleanup.|
|`14c8: ret`|Returns to the caller|Returns the value stored in `eax`.|

**Important observations:**

- `jns` means _jump if not sign_: it branches when the sign flag is clear.
    
- `jne` means _jump if not equal_: it branches when the zero flag is clear.
    
- All three return paths converge at the same function epilogue (`14c3`).
    
- The return value is placed in `eax` before the epilogue executes.
    

## 6. C reconstruction

My reconstruction:

```
int main()
{
    int a = 0xFFFFFFFB;

    if(a < 0)
    {
        return 1;
    }

    if(a == 0)
    {
        return 2;
    }

    return 3;
}
```

**Interpretation:**

- `0xFFFFFFFB` is the 32-bit hexadecimal representation of `-5` in two's complement.
    
- `a < 0` is true, so the program returns `1`.
    
- The second condition and final return are unreachable for this initial value.
    

My reconstruction is behaviorally equivalent to the original.

## 7. Verification

- **Does it compile?** Yes.
    
- **Does it behave like the original?** Yes. Both return `1`.
    
- **Which test cases did I try?**
    
    - `-5` → returns `1`.
        
    - `0` → returns `2`.
        
    - `5` → returns `3`.
        
- **What differences did I discover?**
    
    - The original uses decimal `-5`; my reconstruction uses hexadecimal `0xFFFFFFFB`.
        
    - Both represent the same 32-bit signed integer.
        

## 8. Things to look out for

- A hexadecimal value can represent a negative signed integer when interpreted using two's complement.
    
- `jns` does not directly mean `value >= 0`; it checks the processor's sign flag after the comparison.
    
- Multiple early returns can share one function epilogue instead of having separate `ret` instructions.

## 9. Key RE lesson

Early returns are commonly implemented by setting a return value in `eax` and jumping to a shared function epilogue. Conditional jumps determine which return path executes, while unconditional jumps prevent execution from falling through into other return paths. During reverse engineering, identify the conditions, return values, and common exit point to reconstruct the original control flow.

## 10. Checkpoint

- I can explain every relevant instruction.
    
- I can reconstruct the program without the source.
    
- I can explain why my reconstruction is equivalent.
    
- I can solve a slightly modified version independently.