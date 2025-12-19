when we compile a C program, they all go through a phases
starting from the source code .c file, they're then converted into assembly code, and later to an object file using an assembler

now, this object file contains interesting data. It holds the initialized, and unitialized global values, the literal machine instructions to run
when the program is executed, and more.

This object file several headers in it, one of which is .text, which contains all the program machine instruction

Now, when I say maching instruction, it doesn't mean `int x = 0` or something like that. these maching instrucitons are as fundamental as they can be, executing 
a very specific task in sequence, manipulating the underlying hardware of the computer.

On a high level, we know that we have variables, functions and more. But these things are nothing but addresses in the program's virtual memory during execution.
all data structures in the code are just an array of bytes(yes, everything in C you see is represented in array of bytes)