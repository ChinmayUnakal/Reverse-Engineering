# Program 08 — Bitwise Operators

## 1. Objective

Learn how bitwise operators (`&`, `|`, `^`) operate on individual bits and translate into x86-64 assembly instructions.

## 2. Original C

```
int main(void)
{
    int a = 0x55;
    int b = 0x0F;

    int x = a & b;
    int y = a | b;
    int z = a ^ b;

    return x + y + z;
}
```

## 3. Behavioral prediction

- **Expected output:** None.
    
- **Expected return value:** `95` (`0x5F`).
    
- **Important variable values:**
    
    - `a = 0x55` = `85`
        
    - `b = 0x0F` = `15`
        
    - `x = 0x05` = `5`
        
    - `y = 0x5F` = `95`
        
    - `z = 0x5A` = `90`
        
- **Expected operations:** AND, OR, XOR, followed by addition.
    

## 4. Compilation

- **Compiler/version:** GCC 15.2.0 (MSYS2).
    
- **Architecture:** x86-64 Windows.
    
- **Build configuration:** `-O0 -g`.
    
- **Compilation command:**
    

```
gcc -O0 -g -o bitwise.exe 08_bitwise_operators.c
```

## 5. Assembly observations

|Instruction|Register or memory effect|Interpretation|
|---|---|---|
|`mov [rbp-4], 55h`|Stores `0x55`|Initializes `a`|
|`mov [rbp-8], 0Fh`|Stores `0x0F`|Initializes `b`|
|`mov eax, [rbp-4]`|Loads `a` into `EAX`|Prepares first operand|
|`and eax, [rbp-8]`|Bitwise AND|Calculates `x`|
|`or eax, [rbp-8]`|Bitwise OR|Calculates `y`|
|`xor eax, [rbp-8]`|Bitwise XOR|Calculates `z`|
|`mov [rbp-0Ch], eax`|Stores the result|Saves `x`|
|`mov [rbp-10h], eax`|Stores the result|Saves `y`|
|`mov [rbp-14h], eax`|Stores the result|Saves `z`|
|`add edx, eax` / `add eax, edx`|Adds intermediate values|Calculates `x + y + z`|
|`ret`|Returns to caller|Final result is in `EAX`|

### Bit-level calculation

```
a = 0x55 = 0101 0101
b = 0x0F = 0000 1111
```

- **AND (**`**&**`**):** A bit is `1` only if both bits are `1`. Result: `0000 0101` = `5`.
    
- **OR (**`**|**`**):** A bit is `1` if either bit is `1`. Result: `0101 1111` = `95`.
    
- **XOR (**`**^**`**):** A bit is `1` if the bits differ. Result: `0101 1010` = `90`.
    

Final return value: `5 + 95 + 90 = 190`.

## 6. C reconstruction

**Status: Not independently verified.** The reconstruction below matches the observed assembly's operations.

```
int main(void)
{
    int a = 0x55;
    int b = 0x0F;

    int x = a & b;
    int y = a | b;
    int z = a ^ b;

    return x + y + z;
}
```

**Correction to the prediction:** The correct return value is **190**, not 95. The sum is `5 + 95 + 90 = 190` (`0xBE`).

## 7. Verification

- **Does it compile?** Compile using the command in Section 4.
    
- **Expected return value:** `190`.
    
- **Test:** In PowerShell, run `.\bitwise.exe`, then check `$LASTEXITCODE`. Expected: `190`.
    
- **Difference discovered:** Bitwise operators manipulate individual bits; they are different from logical operators such as `&&` and `||`.
    

## 8. What I misunderstood

Record any confusion between bitwise and logical operators. In particular, `&` operates on bits, while `&&` evaluates truth values.

## 9. Key RE lesson

The `AND`, `OR`, and `XOR` instructions perform bitwise operations directly on their operands. Track which registers contain the operands and where each result is stored. Unlike logical expressions, bitwise operations can produce values other than `0` and `1`.

## 10. Checkpoint

- I can explain `AND`, `OR`, and `XOR` at the bit level.
    
- I can calculate their results by hand.
    
- I can trace the result through `EAX` and the stack variables.
    
- I can independently solve the same program with different input values.