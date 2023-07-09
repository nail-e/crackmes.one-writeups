###### nail_
## Preface
---
This CTF task isn't the most complicated CTF and has quite a unique way to be solved. We are given the tip: "*What is the num less than 0?*" which we should keep in mind through solving the task.

### Required Tools:
- Ghidra (Not needed for the flag)

## Recon
---
On executing the file, we are greeted with a menu:
![[Pasted image 20230624155227.png]]

Assuming we attempt to claim the flag instantly, we are greeted with:
![[Pasted image 20230624160545.png]]

If we attempt to claim either Coca-Cola or Pepsi, we are prompted to input an amount. Assuming we love Coke and buy 2, we end up with 0 coins.
![[Pasted image 20230624160739.png]]

Now, assuming we're indebted to Coke and it's army of sugar and coca leaves, we want to return a can of coke, inputting -1 cokes.
![[Pasted image 20230624170733.png]]

Alas! We have 15 coins and 11 drinks! But how is this?

This is easily explainable:
Assuming $x$ is our input, $I$ is our initial value and $y$ is our final value, $$I-x = y$$
If $x = -1$ and $I = 10$, this can be substituted as $$10 --1 = y$$ where $y = 11$


The same logic can be applied for the coins:
Assuming $x$ is our requested items, $I$ is our initial value and $P$ is our price and $y$ is our final value,  $$I - (Px) = y$$
If $x = -1$, $I = 10$, $P = 5$, this can be substituted as $$10 - (5 \times-1) = y$$ where $y = 15$ 

Thus, we can substitute $y = 100$ for the coin equation to find $x$, $$10 - (5 \times x) = 100$$ where $x = -18$

Inputting "-18" gives us 100 coins, allowing us to obtain the flag. `flag{v3nd1nd_m4ch1n3_1s_4w3s0m3}`
![[Pasted image 20230624172454.png]]

## Further Analysis
---
Inputting "-18" is not the only way to form an overflow. The following is the overthinker's method:

Using Ghidra's Code browser, extracting the `vending_machine` file into a folder and placing the file in Ghidra the decompiled file in assembly is generated.
![[Pasted image 20230624161453.png]]
We see references to "Elf64" in the assembly code, suggesting that the code supports a 64-bit instruction set (which is exclusive to UNIX systems, but this is irrelevant). Keep this in mind for later.

Looking at the symbol tree, we see reference to Rust (based) and `rustc` compiler functions.
![[Pasted image 20230624162123.png]]
A simple scan through of *The Rust Programming Language* section on data types shows that Rust contains 6 separate lengths of integer data, either signed or unsigned.

![[Pasted image 20230624163930.png]]
Recalling that the code supports a 64-bit instructions set and that it's possible to obtain unsigned integers we can use from signed 8 to 64 bit integers in order to find the most efficient way to cause an integer overflow.

The largest integer per bit set is given as:

| Length | Amount of Values | Highest Integer   |
| ------ | ---------------- | ----------------- |
| 8-bit  | 2^8              | 255               |
| 16-bit | 2^16             | 65,535            |
| 32-bit | 2^32             | 4,294,967,295     |
| 64-bit | 2^64             | 1.844674407x10^19 |
As 64 bit is way too inefficient to input so it's either 255, 65,535 or 4,294,967,295.

As 5 coins are removed from us each time we order something, to find the value we would need to divide the integer limit by 5.

#### 8-bit
$$255 \div 5 = 51$$
Inputting this as the amount, we get
![[Pasted image 20230624172914.png]]

Which doesn't hold up.

#### 16-bit
$$65535 \div 5 = 13107$$
Inputting this as the amount, we get
![[Pasted image 20230624173034.png]]

Which doesn't hold up.

#### 32-bit
$$4294967295 \div 5 = 858993459$$
Inputting this as the amount, we get
![[Pasted image 20230624173401.png]]

Which doesn't hold up BUT it somehow goes to 11. This suggests the limit has been hit prior. Using the formula  $$I - (Px) = y$$we substitute values to get $$10 - (5x) = 0$$ where $x = 2$

Dividing $858993459$ by $2$, we get $429496729.5$
Rounding $429496729.5$ to the nearest integer, we get $429496730$
Adding $1$ due to finding the maximum value of 32 bit is $2^{32} - 1$ is $429496731$
Adding $1$ due to 2's complement, we get $429496732$

If we input $429496732$, we get 429,496,732 coins. Enough for a flag.
![[Pasted image 20230624174046.png]]

