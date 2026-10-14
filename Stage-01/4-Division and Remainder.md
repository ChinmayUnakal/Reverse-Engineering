# Program 04 — Division and Remainder

## 1. Objective

This program teaches me to identify integer division and remainder operations in x86-64 assembly.

- Recognize `CDQ` and `IDIV`.
    
- Understand how division produces both a quotient and a remainder.
    
- Track values through `EAX` and `EDX`.
    
- Reconstruct C division and modulo expressions from assembly.
    

## 2. Original C

```
int main(void)
{
    int a = 23;
    int b = 5;

    int quotient = a / b;
    int remainder = a % b;

    return quotient + remainder;
}
```

## 3. Behavioral Prediction

- **Expected output:** No output; the program does not print anything.
    
- **Expected return value:** `7`
    
- **Important variable values:**
    
    - `a = 23`
        
    - `b = 5`
        
    - `quotient = 23 / 5 = 4`
        
    - `remainder = 23 % 5 = 3`
        
    - Return value = `4 + 3 = 7`
        
- **Expected branches or function calls:** GCC runtime helper `__main`; no conditional branches.
    

## 4. Compilation

- **Compiler:** GCC 15.2.0 (MSYS2)
    
- **Architecture:** Windows x86-64
    
- **Build configuration:** `-O0 -g`
    
- **Compilation command:**
    

```
gcc -O0 -g -o div.exe 04_division_remainder.c
```

## 5. Assembly Observations

|Instruction|Register or memory effect|Interpretation|
|---|---|---|
|`mov [rbp-4], 17h`|Stores 23|Initializes `a`. `17h` = 23 decimal.|
|`mov [rbp-8], 5`|Stores 5|Initializes `b`.|
|`mov eax, [rbp-4]`|`EAX = 23`|Loads the dividend.|
|`cdq`|Sign-extends `EAX` into `EDX:EAX`|Prepares the signed dividend for 32-bit division.|
|`idiv dword ptr [rbp-8]`|Divides `EDX:EAX` by 5|Quotient goes into `EAX`; remainder goes into `EDX`.|
|`mov [rbp-0Ch], eax`|Stores 4|Saves the quotient.|
|Second `mov eax, [rbp-4]`|`EAX = 23`|Reloads the dividend for another division.|
|Second `cdq` and `idiv`|Quotient = 4; remainder = 3|Computes the remainder again.|
|`mov [rbp-10h], edx`|Stores 3|Saves the remainder.|
|`mov edx, [rbp-0Ch]`|`EDX = 4`|Loads the quotient.|
|`mov eax, [rbp-10h]`|`EAX = 3`|Loads the remainder.|
|`add eax, edx`|`EAX = 7`|Adds quotient and remainder.|
|`add rsp, 30h; pop rbp; ret`|Restores stack and returns|Returns 7 in `EAX`.|

### Key concept: `CDQ` and `IDIV`

```
mov eax, dword ptr [rbp-4]
cdq
idiv dword ptr [rbp-8]
```

For signed 32-bit division:

- `CDQ` sign-extends `EAX` into the 64-bit dividend represented by `EDX:EAX`.
    
- `IDIV` divides that dividend by the specified operand.
    
- The quotient is stored in `EAX`.
    
- The remainder is stored in `EDX`.
    

For this example:

```
23 / 5 = 4 remainder 3
EAX = 4
EDX = 3
```

**Important observation:** The compiler performs division twice—once for the quotient and once for the remainder. It reloads `a` and executes `IDIV` again because the quotient and remainder are stored in different variables.

## 6. C Reconstruction

```
int main(void)
{
    int a = 23;
    int b = 5;

    int quo = a / b;
    int rem = a % b;

    return quo + rem;
}
```

My reconstruction is behaviorally equivalent to the original. The variable names differ, but that does not change the program's behavior.

## 7. Verification

- **Does it compile?** Expected to compile successfully.
    
- **Does it behave like the original?** Yes, if it returns 7.
    
- **Test case:** `23 / 5` gives quotient 4 and remainder 3.
    
- **Differences discovered:** No meaningful behavioral differences. The program returns 7 but prints nothing.
    

Verify the exit status in PowerShell:

```
.\div.exe
$LASTEXITCODE
```

Expected exit status: `7`.

## 8. Watch out for these

- Division produces two results: quotient in `EAX` and remainder in `EDX`.
    
- `CDQ` prepares the signed dividend; it does not perform division itself.
    
- `IDIV` is signed division, while `DIV` is unsigned division.
    
- The program's return value is not automatically printed.


## 9. Key RE Lesson

`IDIV` produces both a quotient and a remainder. For 32-bit signed division, the dividend is held in `EDX:EAX`, the divisor is the instruction's operand, the quotient goes to `EAX`, and the remainder goes to `EDX`. Tracking these registers allows me to reconstruct both `/` and `%` operations in C.

## 10. Checkpoint

- I can explain `CDQ` and `IDIV`.
    
- I can identify the quotient in `EAX` and remainder in `EDX`.
    
- I can reconstruct the division and modulo operations without the original source.
    
- I can explain why the program returns 7 without printing anything.
    
- I can solve a modified division example independently.


