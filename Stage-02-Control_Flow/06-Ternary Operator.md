# Program 06 — Ternary Operator

## 1. Objective

Understand how the C ternary operator (`?:`) is translated into x86-64 assembly. Learn to recognize conditional comparisons and conditional moves, and reconstruct selection logic without relying on explicit conditional jumps.

## 2. Original C

```
int main(void)
{
    int a = 12;
    int b = 7;

    int result = (a > b) ? a : b;

    return result;
}
```

## 3. Behavioral Prediction

- **Expected output:** None (no printing).
    
- **Expected return value:** 12.
    
- **Important variable values:**
    
    - `a = 12`
        
    - `b = 7`
        
    - `result = 12`
        
- **Expected branches or function calls:** A comparison between `a` and `b`, followed by selection of the greater value.
    

## 4. Compilation

- **Compiler/version:** GCC 15.2.0 (MSYS2).
    
- **Architecture:** x86-64 Windows.
    
- **Build configuration:** `-O0 -g`.
    
- **Compilation command:**
    

```
gcc -O0 -g -o 06-tern.exe 06-ternary.c
```

## 5. Assembly Observations

|Instruction|Register or memory effect|Interpretation|
|---|---|---|
|`mov dword ptr [rbp-4], 0Ch`|Stores 12|Initializes `a`.|
|`mov dword ptr [rbp-8], 7`|Stores 7|Initializes `b`.|
|`mov edx, [rbp-8]`|`EDX = 7`|Loads `b`.|
|`mov eax, [rbp-4]`|`EAX = 12`|Loads `a`.|
|`cmp edx, eax`|Compares 7 with 12|Sets CPU flags based on `b - a`.|
|`cmovge eax, edx`|Conditional move|Replaces `EAX` with `EDX` if `b >= a`.|
|`mov [rbp-0Ch], eax`|Stores 12|Saves the selected value in `result`.|
|`mov eax, [rbp-0Ch]`|`EAX = 12`|Loads the return value.|
|`ret`|Returns from function|Returns the value in `EAX`.|

### Important instruction: `cmovge`

`cmovge` means **Conditional Move if Greater or Equal (signed comparison)**.

In this program:

1. `cmp edx, eax` compares `b` against `a`.
    
2. If `b >= a`, `cmovge` copies `EDX` into `EAX`.
    
3. Otherwise, `EAX` retains its original value (`a`).
    

Effectively:

```
result = (b >= a) ? b : a;
```

Notice that GCC reversed the comparison relative to the original C expression while preserving its behavior.

## 6. C Reconstruction

```
int main(void)
{
    int a = 12;
    int b = 7;

    int c = (b >= a) ? b : a;

    return c;
}
```

This is logically equivalent to the original program.

## 7. Verification

- **Does it compile?** Yes.
    
- **Does it behave like the original?** Yes.
    
- **Test cases:**
    
    - `a = 12, b = 7` → returns 12.
        
    - `a = 7, b = 12` → returns 12.
        
    - `a = 10, b = 10` → returns 10.
        
- **Differences discovered:** Variable names and comparison direction differ, but the selection logic is equivalent.

## 8. What I Misunderstood

- Initially, I expected a conditional jump such as `jge` or `jle`.
    
- Instead, GCC used `cmovge`, which conditionally updates a register without changing the instruction flow through a branch.
    
- The comparison direction was reversed because GCC selected between the two values using `b >= a`.

## 9. Key RE Lesson

The ternary operator does not necessarily translate into conditional jumps. Compilers can use conditional move instructions such as `cmovge` to select between values based on CPU flags.

When reconstructing C from assembly, pay attention to the operands of `cmp`, the condition used by `cmov`, and which register retains its original value when the condition is false.

**A conditional move can implement selection logic without an explicit branch.**

## 10. Checkpoint

- Identified the comparison instruction (`cmp`).
    
- Identified the conditional move (`cmovge`).
    
- Reconstructed the selection logic.
    
- Verified logical equivalence.
    
- Independently reconstruct a modified version using different values and comparison operators.