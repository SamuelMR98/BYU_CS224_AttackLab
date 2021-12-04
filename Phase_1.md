
Phase 1 is the easiest of the 5. What you are trying to do is overflow the stack with the exploit string and change the return address of getbuf function to the address of touch1 function. You are trying to call the function touch1.

run ctarget executable in gdb and set a breakpoint at getbuf

