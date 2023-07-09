---
tag: Writeup
---
## Preface
---
The first Unix CrackMe on crackmes.one, the entry is crackable and can be finished in 5 minutes or less, even for beginners.   

### Required Tools:
- Ghidra

## Recon
---
Using Ghidra's Code browser, extracting the `level1` file into a folder and placing the file in Ghidra the decompiled file in assembly is generated.
![[Pasted image 20230624135237.png]]

First, we attempt to find an undefined function. The first given undefined function is `checkPass()`. Through inference, it can be suggested that this function checks the input code's validity.
![[Pasted image 20230624135754.png]]

By using CheckPass (Ctrl + E), a C pseudocode listing of the function is generated.
```C
char checkPass(char *param_1)

{
  char cVar1;
  
  if (*param_1 == 's') {
    cVar1 = param_1[1];
    if ((((cVar1 == 'u') && (cVar1 = param_1[2], cVar1 == 'd')) &&
        (cVar1 = param_1[3], cVar1 == 'o')) &&
       (((cVar1 = param_1[4], cVar1 == '0' && (cVar1 = param_1[5], cVar1 == 'x')) &&
        ((cVar1 = param_1[6], cVar1 == '1' && (cVar1 = param_1[7], cVar1 == '8')))))) {
      cVar1 = '\x01';
    }
  }
  else {
    cVar1 = '\0';
  }
  return cVar1;
}
```

The code checks is the given input with char data type, `param_1` contains is first equal to "s" , which then passes the input into `cVar1`'s 1st index if true. A nested if statement checks if `cVar1` is equal to "u" and the 2nd index is equal to "d". The nested if statement continuously checks if the succeeding input follows the string `sudo0x18`. 

We can get the resulting code input as `sudo0x18`, the submitter's username.
![[Pasted image 20230624140839.png]]
## Further Analysis
---
The C code contains logical errors. The first if statement:
```C
if (*param_1 == 's') {
    cVar1 = param_1[1];
```
creates an error where every succeeding input does not affect the results as input into `cVar1` is already passed and unchaged. 
![[Pasted image 20230624141444.png]]