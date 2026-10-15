# Program 07 — Boolean Logic

## 1. Objective

Learn how C logical operators (`&&`, `||`, `!`) translate into comparisons, conditional jumps, and Boolean values (`0` or `1`) in assembly.

## 2. Original C

```
int main(void)
{
    int a = 1;
    int b = 0;

    int x = a && b;
    int y = a || b;
    int z = !a;

    return x + y + z;
}
```

## 3. Behavioral prediction

- **Expected output:** None.
    
- **Expected return value:** `1`.
    
- **Variable values:** `a = 1`, `b = 0`, `x = 0`, `y = 1`, `z = 0`.
    
- **Expected branches:** Conditional jumps implement `&&` and `||`; `SETE` implements `!a`.
    

## 4. Compilation

- **Compiler/version:** GCC 15.2.0 (MSYS2).
    
- **Architecture:** x86-64 Windows.
    
- **Build configuration:** `-O0 -g`.
    
- **Compilation command:**
    

```
gcc -O0 -g -o bool.exe 07_boolean_logic.c
```

## 5. Assembly observations

| Instruction                 | Meaning                                                               |     |     |
| --------------------------- | --------------------------------------------------------------------- | --- | --- |
| `mov [rbp-4], 1`            | Stores `a = 1`.                                                       |     |     |
| `mov [rbp-8], 0`            | Stores `b = 0`.                                                       |     |     |
| `cmp [rbp-4], 0`            | Checks whether `a` is zero.                                           |     |     |
| `je ...`                    | Jumps when the compared value is zero. Used to evaluate `&&` and `    |     |     |
| `mov eax, 1` / `mov eax, 0` | Produces a Boolean result.                                            |     |     |
| `sete al`                   | Sets `AL` to `1` if the previous comparison was equal, otherwise `0`. |     |     |
| `movzx eax, al`             | Zero-extends the one-byte Boolean result into `EAX`.                  |     |     |
| `add edx, eax`              | Adds intermediate results together.                                   |     |     |

### Trace the three expressions

- `x = a && b`**:** `a` is nonzero, but `b` is zero. Result: `x = 0`.
    
- `y = a || b`**:** `a` is already nonzero, so the OR expression is true. Result: `y = 1`.
    
- `z = !a`**:** `a` is nonzero, so `SETE` produces `0`. Result: `z = 0`.
    

Final calculation: `0 + 1 + 0 = 1`.

## 6. C reconstruction

**Status: Not reconstructed independently.**

I found this program confusing and couldn't confidently reconstruct the C from the assembly.

### Correct reconstruction — study only after attempting it

```
int main(void)
{
    int a = 1;
    int b = 0;

    int x = a && b;
    int y = a || b;
    int z = !a;

    return x + y + z;
}
```

### What I need to understand

- How the jumps implement logical AND (`&&`).
    
- How the jumps implement logical OR (`||`).
    
- How `SETE` and `MOVZX` implement logical NOT (`!`).
    
- How the intermediate results are stored and added to produce the return value.

## 7. Verification

- **Does it compile?** Compile using the command in Section 4.
    
- **Expected return value:** `1`.
    
- **Test:** Run `.\bool.exe`, then check `$LASTEXITCODE` in PowerShell. Expected: `1`.
    
- **Difference discovered:** Logical expressions can compile into several instructions and branches rather than one dedicated logical instruction.
    

## 8. What I misunderstood

The confusing part is that `JE` and `JNE` control which Boolean result is produced. Follow each jump to its destination to determine whether the result becomes `0` or `1`.

## 9. Key RE lesson

In C, zero represents false and any nonzero integer represents true in logical expressions. `&&` requires both operands to be true, while `||` requires at least one; `!` reverses truth. Assembly implements these rules through comparisons, jumps, and instructions such as `SETE`.

## 10. Checkpoint

- I can explain how `&&`, `||`, and `!` behave.
    
- I can trace `JE`, `JNE`, and `SETE`.
    
- I can calculate the final return value.
    
- I can independently solve the modified case `a = 0`, `b = 1`.