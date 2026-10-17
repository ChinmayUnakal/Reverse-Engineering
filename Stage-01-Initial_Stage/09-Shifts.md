# Program 09 — Shifts

## 1. Objective

Learn how left and right bit shifts translate into assembly instructions and how to track their results through registers and stack variables.

## 2. Original C

```
int main(void)
{
    int a = 0x80;

    int x = a << 2;
    int y = a >> 2;

    return x + y;
}
```

## 3. Behavioral prediction

- **Expected output:** None.
    
- **Expected return value:** `544`.
    
- **Important variable values:**
    
    - `a = 0x80` = `128`
        
    - `x = 128 << 2` = `512`
        
    - `y = 128 >> 2` = `32`
        
- **Expected branches or function calls:** No conditional branches. The program shifts, stores, adds, and returns.
    

## 4. Compilation

- **Compiler/version:** GCC 15.2.0 (MSYS2).
    
- **Architecture:** x86-64 Windows.
    
- **Build configuration:** `-O0 -g`.
    
- **Compilation command:**
    

```
gcc -O0 -g -o shifts.exe 09_shifts.c
```

## 5. Assembly observations

|Instruction|Register or memory effect|My interpretation|
|---|---|---|
|`mov [rbp-4], 80h`|Stores `128`|Initializes `a`|
|`mov eax, [rbp-4]`|Loads `a` into `EAX`|Prepares the value|
|`shl eax, 2`|Shifts bits left by 2|Calculates `x = a << 2`|
|`mov [rbp-8], eax`|Stores the result|Saves `x = 512`|
|`mov eax, [rbp-4]`|Loads `a` again|Prepares the second shift|
|`sar eax, 2`|Arithmetic-shifts right by 2|Calculates `y = a >> 2`|
|`mov [rbp-0Ch], eax`|Stores the result|Saves `y = 32`|
|`mov edx, [rbp-8]`|Loads `x` into `EDX`|Prepares addition|
|`mov eax, [rbp-0Ch]`|Loads `y` into `EAX`|Prepares addition|
|`add eax, edx`|Adds `x` and `y`|Final result: `544`|

### Understand the shifts

```
a = 128 = 0000 0000 1000 0000

SHL by 2:  0010 0000 0000 = 512
SAR by 2:  0000 0000 0010 0000 = 32
```

- `SHL` shifts bits left, multiplying this positive value by (2^2 = 4).
    
- `SAR` shifts bits right while preserving the sign bit. For this positive value, the result is `32`.
    

**Important:** `SAR` is an arithmetic right shift. `SHR` is a logical right shift, which fills the leftmost bits with zeros.

## 6. C reconstruction

My reconstruction:

```
int main()
{
    int a = 0x80;

    int b = a << 2;
    int c = a >> 2;

    return x + y;
}
```

The shift operations and overall program logic were correctly identified. Only the variable names were inconsistent.

## 7. Verification

- **Does it compile?** Compile using the command in Section 4.
    
- **Expected return value:** `544`.
    
- **Test:** Run `.\shifts.exe`, then check `$LASTEXITCODE` in PowerShell.
    
- **Difference discovered:** The assembly uses `SAR` for the right shift, rather than `SHR`.

## 8. Key RE lesson

`SHL` and `SAR` move bits left and right, respectively. The shift count is `2`, meaning two bit positions. To reconstruct the C, track the input value, shift instruction, result storage location, and final addition.

## 9. Checkpoint

- I can explain `SHL` and `SAR`.
    
- I can calculate both shift results manually.
    
- I can map `[rbp-4]`, `[rbp-8]`, and `[rbp-0Ch]` to variables.
    
- I can independently predict the result if the shift count changes to `3`.