# Program 12 — Mixed Integer Types

## 1. Objective

Learn how signed and unsigned 16-bit integers are converted to 32-bit integers. Use `MOVSX` and `MOVZX` to infer signedness during reverse engineering.

## 2. Original C

```
int main(void)
{
    short a = -100;
    unsigned short b = 500;

    int x = a;
    int y = b;

    return x + y;
}
```

## 3. Behavioral prediction

- **Expected output:** None.
    
- **Expected return value:** `400`.
    
- **Important variable values:**
    
    - `a = -100` → 16-bit representation `0xFF9C`
        
    - `b = 500` → 16-bit representation `0x01F4`
        
    - `x = -100`
        
    - `y = 500`
        
    - `x + y = 400`
        
- **Expected branches or function calls:** No conditional branches. GCC's runtime helper `__main` is called, followed by the conversions and addition.
    

## 4. Compilation

- **Compiler/version:** GCC 15.2.0 (MSYS2).
    
- **Architecture:** x86-64 Windows.
    
- **Build configuration:** `-O0 -g` (unoptimized, with debugging information).
    
- **Compilation command:**
    

```
gcc -O0 -g -o mix_int_typ.exe 12_mixed_integer_types.c
```

## 5. Assembly observations

|Instruction/address|Register or memory effect|My interpretation|
|---|---|---|
|`push rbp`|Saves the previous frame pointer|Function prologue|
|`mov rbp, rsp`|Establishes the stack frame|Sets the frame pointer|
|`sub rsp, 30h`|Reserves stack space|Allocates the stack frame|
|`call __main`|Calls GCC's runtime helper|Compiler-generated initialization|
|`mov word ptr [rbp-2], 0FF9Ch`|Stores the 16-bit value `0xFF9C`|Initializes `a = -100`|
|`mov word ptr [rbp-4], 1F4h`|Stores the 16-bit value `0x01F4`|Initializes `b = 500`|
|`movsx eax, word ptr [rbp-2]`|Sign-extends the 16-bit value to 32 bits|Converts `a` to `int`, preserving `-100`|
|`mov dword ptr [rbp-8], eax`|Stores the 32-bit result|Saves `x = -100`|
|`movzx eax, word ptr [rbp-4]`|Zero-extends the 16-bit value to 32 bits|Converts `b` to `int`, producing `500`|
|`mov dword ptr [rbp-0Ch], eax`|Stores the 32-bit result|Saves `y = 500`|
|`mov edx, dword ptr [rbp-8]`|Loads `x` into `EDX`|Prepares the first operand|
|`mov eax, dword ptr [rbp-0Ch]`|Loads `y` into `EAX`|Prepares the second operand|
|`add eax, edx`|Computes `y + x`|Result is `400`, returned in `EAX`|
|`add rsp, 30h`|Releases stack space|Function epilogue|
|`pop rbp`|Restores the previous frame pointer|Function epilogue|
|`ret`|Returns to the caller|Returns `400` in `EAX`|

### Understanding the conversions

**Signed** `short`**:**

The 16-bit pattern for `-100` is `0xFF9C`. `MOVSX` sign-extends it:

```
16-bit:  0xFF9C
32-bit:  0xFFFFFF9C
Decimal: -100
```

**Unsigned** `**short**`**:**

The 16-bit pattern for `500` is `0x01F4`. `MOVZX` zero-extends it:

```
16-bit:  0x01F4
32-bit:  0x000001F4
Decimal: 500
```

The final addition is:

```
-100 + 500 = 400
```

## 6. C reconstruction

**Status: Partially reconstructed.** I identified the conversion and addition structure, but the declaration of `b` loses its unsigned type. My declaration of `y` as `unsigned int` also differs from the original, although it still produces the same numerical value for this particular input.

Your reconstruction:

```
int main()
{
    short a = -100;
    short b = 500;

    int x = a;
    unsigned int y = b;

    return x + y;
}
```

Corrected reconstruction:

```
int main(void)
{
    short a = -100;
    unsigned short b = 500;

    int x = a;
    int y = b;

    return x + y;
}
```

## 7. Verification

- **Does it compile?** Check by compiling the corrected reconstruction.
    
- **Does it behave like the original?** It should return `400`.
    
- **Which test cases did I try?** Record the actual tests after running them.
    
- **What differences did I discover?** `b` must be `unsigned short` to match the original source-level type. The assembly's zero extension is consistent with that declaration.
    

To verify the return value in PowerShell:

```
.\mix_int_typ.exe
$LASTEXITCODE
```

Expected exit code: `144` on Windows, because process exit codes expose only the low 8 bits of the returned value (`400 mod 256 = 144`). The CPU-level return value is still `400` in `EAX`.

## 8. What I misunderstood

I declared `b` as `short`, which is signed, even though the original uses `unsigned short`. I also changed `y` to `unsigned int`, although the original declares it as `int`.

For this input, both versions can produce the same final arithmetic result. That does not make their types equivalent in general. The distinction matters for values outside the signed `short` range.

## 9. Key RE lesson

`MOVSX` sign-extends a narrow value, while `MOVZX` zero-extends it. These instructions help infer how the program interprets 16-bit values when converting them to 32-bit integers. However, assembly reveals behavior more directly than it reveals exact source declarations, so confirm your reconstruction against the operations and values rather than relying on a single instruction alone.

## 10. Checkpoint

- I can explain `MOVSX` versus `MOVZX` for 16-bit operands.
    
- I can derive the 32-bit representation of `-100` and `500`.
    
- I can explain why the CPU-level return value is `400`.
    
- I understand why the Windows process exit code is `144`.
    
- I can reconstruct signed and unsigned conversions independently.