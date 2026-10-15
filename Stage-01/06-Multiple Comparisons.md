# Program 06 — Multiple Comparisons

## 1. Objective

Learn how multiple `if` statements are translated into comparisons, conditional jumps, and different return paths in assembly.

## 2. Original C

```
int main(void)
{
    int a = 10;

    if (a == 10)
        return 1;

    if (a != 20)
        return 2;

    return 0;
}
```

## 3. Behavioral prediction

- **Expected output:** None.
    
- **Expected return value:** `1`.
    
- **Important variable values:** `a = 10`.
    
- **Expected branches or function calls:** The first comparison succeeds, so the program returns `1` immediately. The second comparison is never executed.
    

## 4. Compilation

- **Compiler/version:** GCC 15.2.0 (MSYS2)
    
- **Architecture:** x86-64 Windows.
    
- **Build configuration:** `-O0 -g`.
    
- **Compilation command:**
    

```
gcc -O0 -g -o mul_com.exe 06_multiple_comparisons.c
```

## 5. Assembly observations

|Instruction|Register or memory effect|My interpretation|
|---|---|---|
|`push rbp`|Saves the previous frame pointer|Function prologue|
|`mov rbp, rsp`|Establishes the stack frame|Sets up local-variable addressing|
|`sub rsp, 30h`|Reserves stack space|Allocates stack space for the function|
|`mov [rbp-4], 0Ah`|Stores `10` in the local variable slot|`a = 10`|
|`cmp dword ptr [rbp-4], 0Ah`|Compares `a` with `10`|Tests `a == 10`|
|`jne main+0x21`|Jumps if the values are not equal|Skips `return 1` if the first condition is false|
|`mov eax, 1`|Places `1` in the return register|Returns `1`|
|`jmp main+0x33`|Jumps to the function epilogue|Skips the remaining conditions|
|`cmp dword ptr [rbp-4], 14h`|Compares `a` with hexadecimal `14` (`20`)|Tests whether `a == 20`|
|`je main+0x2e`|Jumps if the values are equal|Skips `return 2` and selects `return 0`|
|`mov eax, 2`|Places `2` in `EAX`|Returns `2`|
|`jmp main+0x33`|Jumps to the function epilogue|Skips the final return assignment|
|`mov eax, 0`|Places `0` in `EAX`|Returns `0`|
|`add rsp, 30h`|Releases stack space|Function epilogue|
|`pop rbp`|Restores the previous frame pointer|Restores the caller's frame|
|`ret`|Returns to the caller|Ends the function|

### Important instruction details

- `CMP` updates CPU flags as if it subtracts the operands, without saving the subtraction result.
    
- `JNE` means **jump if not equal**.
    
- `JE` means **jump if equal**.
    
- `0Ah` is hexadecimal for `10`; `14h` is hexadecimal for `20`.
    
- `EAX` contains the integer return value when `main` returns.
    

### Control flow with `a = 10`

1. Compare `a` with `10`: equal, so `JNE` is **not taken**.
    
2. Execute `mov eax, 1`.
    
3. Jump to the epilogue, skipping the second comparison.
    
4. Return `1`.
    

## 6. C reconstruction

Your reconstruction reversed the second condition's return paths. The assembly actually implements `return 2` when `a != 20`, and `return 0` when `a == 20`.

```
int main(void)
{
    int a = 10;

    if (a == 10)
        return 1;

    if (a == 20)
        return 0;
    else
        return 2;
}
```

This reconstruction is logically equivalent to the original, although the original expresses the second condition as `a != 20`.

## 7. Verification

- **Does it compile?** Compile the reconstruction using the command in Section 4.
    
- **Does it behave like the original?** Yes, if the conditions and return paths are reconstructed as above.
    
- **Which test cases did I try?**

|Value of `a`|First condition|Second condition|Return value|
|---|---|---|---|
|`10`|`a == 10` is true|Not reached|`1`|
|`20`|`a == 10` is false|`a != 20` is false|`0`|
|`30`|`a == 10` is false|`a != 20` is true|`2`|

To test a return value in PowerShell, run the executable and then inspect `$LASTEXITCODE`:

```
.\mul_com.exe
$LASTEXITCODE
```

For the original program, the expected exit code is `1`.

## 8. What I misunderstood

- I changed the second condition from `a != 20` to `a == 20`. That changes the condition's meaning.
    
- I placed the return values on the wrong paths. `JE` jumps to `return 0` when `a == 20`; otherwise, execution reaches `return 2`.
    
- I need to follow the destination of each conditional jump rather than infer the result from the instruction name alone.

## 9. Key RE lesson

Multiple `if` statements become a sequence of comparisons and conditional jumps. A return path can bypass every subsequent comparison, so not every condition necessarily executes. To reconstruct the source accurately, trace both the taken and not-taken paths and identify the value placed in `EAX` on each path.

## 10. Checkpoint

- I can explain `CMP`, `JNE`, and `JE`.
    
- I can reconstruct the program without looking at the original C.
    
- I can explain why each jump leads to a particular return value.
    
- I can independently predict the result if `a` changes to `20` or `30`.