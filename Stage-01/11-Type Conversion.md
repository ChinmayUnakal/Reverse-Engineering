# Program 11 — Type Conversion

## 1. Objective

Learn how signed and unsigned characters are converted to integers, and how `MOVSX` and `MOVZX` reveal the conversion type in assembly.

## 2. Original C

```
int main(void)
{
    char a = -10;
    unsigned char b = 250;

    int x = a;
    int y = b;

    return x + y;
}
```

## 3. Behavioral prediction

- **Expected output:** None.
    
- **Expected return value:** `240`.
    
- **Important variable values:**
    
    - `a = -10` → byte representation `0xF6`
        
    - `b = 250` → byte representation `0xFA`
        
    - `x = -10`
        
    - `y = 250`
        
- **Expected branches or function calls:** No conditional branches. The program converts two byte-sized values to integers and adds them.
    

## 4. Compilation

- **Compiler/version:** GCC 15.2.0 (MSYS2).
    
- **Architecture:** x86-64 Windows.
    
- **Build configuration:** `-O0 -g`.
    
- **Compilation command:**
    

```
gcc -O0 -g -o typ_con.exe 11_type_conversion.c
```

## 5. Assembly observations

|Instruction|Register or memory effect|My interpretation|
|---|---|---|
|`mov byte ptr [rbp-1], 0F6h`|Stores byte `0xF6`|Initializes `a = -10`|
|`mov byte ptr [rbp-2], 0FAh`|Stores byte `0xFA`|Initializes `b = 250`|
|`movsx eax, byte ptr [rbp-1]`|Sign-extends the byte to 32 bits|Converts signed `char` to `int` (`-10`)|
|`mov [rbp-8], eax`|Stores the converted value|Saves `x = -10`|
|`movzx eax, byte ptr [rbp-2]`|Zero-extends the byte to 32 bits|Converts `unsigned char` to `int` (`250`)|
|`mov [rbp-0Ch], eax`|Stores the converted value|Saves `y = 250`|
|`add eax, edx`|Adds `x` and `y`|Final result: `240`|

### Key distinction

- `MOVSX`: Extends the sign bit. `0xF6` becomes `0xFFFFFFF6`, representing `-10` as a 32-bit signed integer.
    
- `MOVZX`: Fills the upper bits with zeros. `0xFA` becomes `0x000000FA`, representing `250`.

## 6. C reconstruction

**Status: Partially reconstructed.** I correctly identified the byte values and the integer conversions, but declared `b` as a signed `char` rather than `unsigned char`. That changes how the byte is interpreted.

Your reconstruction:

```
int main()
{
    char a = 0xF6;
    char b = 0xFA;

    int x = a;
    int y = b;

    return x + y;
}
```

## 7. Verification

- **Does it compile?** Verify using the command in Section 4.
    
- **Expected return value:** `240`.
    
- **Test:** Run `.\typ_con.exe`, then check `$LASTEXITCODE` in PowerShell.
    
- **Difference discovered:** The same byte can represent different integer values depending on whether it is interpreted as signed or unsigned.
    

## 8. What I misunderstood

I used `char` for both variables. The assembly's `MOVZX` indicates that `b` is zero-extended, which is consistent with an unsigned byte. `MOVSX` indicates sign extension for `a`.

## 9. Key RE lesson

When reconstructing type conversions, inspect how narrow values are extended into larger registers. `MOVSX` preserves signed interpretation, while `MOVZX` preserves unsigned interpretation. These instructions can reveal the likely signedness of source variables.

## 10. Checkpoint

- I can explain `MOVSX` versus `MOVZX`.
    
- I can calculate the 32-bit representation of `0xF6` and `0xFA`.
    
- I can explain why the return value is `240`.
    
- I can distinguish signed and unsigned byte conversions from assembly.