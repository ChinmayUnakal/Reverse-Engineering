# Program 10 — Increment and Decrement

## 1. Objective

Learn how increment (`++`) and decrement (`--`) operations translate into arithmetic instructions and how to track changes to a variable.

## 2. Original C

```
int main(void)
{
    int a = 10;

    a++;
    ++a;
    a--;
    --a;

    return a;
}
```

## 3. Behavioral prediction

- **Expected output:** None.
    
- **Expected return value:** `10`.
    
- **Variable values:** `10 → 11 → 12 → 11 → 10`.
    
- **Expected branches:** None. Each operation modifies `a` directly.
    
  I expected it to use INC and DEC but it simply used add and sub.

## 4. Compilation

- **Compiler/version:** GCC 15.2.0 (MSYS2).
    
- **Architecture:** x86-64 Windows.
    
- **Build configuration:** `-O0 -g`.
    
- **Compilation command:**
    

```
gcc -O0 -g -o inc_dec.exe 10_increment_decrement.c
```

## 5. Assembly observations

|Instruction|Register or memory effect|My interpretation|
|---|---|---|
|`mov [rbp-4], 0Ah`|Stores `10`|Initializes `a`|
|`add [rbp-4], 1`|Increments `a` by one|Implements `a++` or `++a`|
|`sub [rbp-4], 1`|Decrements `a` by one|Implements `a--` or `--a`|
|`mov eax, [rbp-4]`|Loads final `a` into `EAX`|Prepares the return value|
|`ret`|Returns to caller|Returns `10`|

The four arithmetic instructions update the same stack variable: `10 + 1 + 1 - 1 - 1 = 10`.

## 6. C reconstruction

**Status: Reconstructed successfully.** Your reconstruction matches the original C.

```
int main(void)
{
    int a = 10;

    a++;
    ++a;
    a--;
    --a;

    return a;
}
```

## 7. Verification

- **Does it compile?** Verify using the command in Section 4.
    
- **Expected return value:** `10`.
    
- **Test:** Run `.\inc_dec.exe`, then check `$LASTEXITCODE` in PowerShell.
    
- **Difference discovered:** Both prefix and postfix forms compile to identical instructions here because their expression values are not used.

## 8. What I misunderstood

No reconstruction mistake identified.

## 9. Key RE lesson

Increment and decrement can appear as `ADD` and `SUB` instructions that modify a memory location directly. Prefix and postfix forms differ when the expression's value is used, but they have the same effect when used alone as statements.

## 10. Checkpoint

- I can identify increment and decrement instructions.
    
- I can track a variable's value after each operation.
    
- I can explain why the return value is `10`.
    
- I can explain when prefix and postfix forms behave differently.