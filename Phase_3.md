# Phase 3

Phase 3 is kinda similar to phase two except that we are trying to call the function touch3 and have to pass our cookie to it as string

In the instruction it tells you that if you store the cookie in the buffer allocated for getbuf, the functions hexmatch and strncmp may overwrite it as they will be pushing data on to the stack, so you have to be careful where you store it.

We will be storing the cookie after touch3.

So let's pass the address for the cookie to register $rdi

The total bytes before the cookie are ```buffer + 8 bytes for return address of rsp + 8 bytes for touch3 0x18 + 8 + 8 = 28 ``` (40 decimnal)

Grab the address for rsp from phase 2: ``0x55617e98`` Add ```0x28``` ```0x55617e98 + 0x28 = 0x55617EC0``` Now you need this assembly code, same steps generating the byte representation

Create the file ```phase3.s```.

```assembly
    movq $0x55620D00,%rdi /* %rsp + 0x18 */
    retq
```
Now you need the byte representation of the code you wrote above, compile it with gcc then dissasemble it.

```ssh
gcc -c phase3.s
objdump -d phase3.o  > phase3.d 
```
