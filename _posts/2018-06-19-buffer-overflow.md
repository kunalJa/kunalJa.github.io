---
layout: post
title: "Buffer Overflow Attack"
share: false
categories:
  - Seed Labs
tags:
  - Security
  - C
  - Summer 2018
---
[Much of this information has been gained from following of SEED labs at Syracuse University.](http://www.cis.syr.edu/~wedu/seed/labs.html)<br/>
It is possible to change the behavior of programs without changing their source code when those programs accept input from a file or stdin.
Many standard functions in the C language that deal with buffers (which we can understand to be an array or some other block of contiguous space in memory) do not restrict operations on those buffers by the buffer’s length. For example, the strcpy(dest, src) function will copy chars from src to dest until it sees a null character. It has no idea the size of the destination buffer and will keep copying bytes over- overwriting information that may exist outside the buffer. This point is the key behind the buffer overflow attack. A malicious entity could not only overwrite useful information but could get the program to execute any code it wants!

The reason this is possible is because of the call stack in a program.
A running process, in RAM, has OS information, the actual lines of code to be executed, static and uninitialized variables, a heap and a stack. The stack holds local variables, returned variables, and variables passed in as arguments. But it also holds the return address of calling functions. The diagram below helps clarify the layout of a general stack:
(http://www.cis.syr.edu/~wedu/seed/Labs_16.04/Software/Buffer_Overflow)

![alt]({{ site.url }}/images/posts/stack_frame.jpg)

[Let’s assume our program is running as a Set-UID process.] <br/>
So, to get the program to change its behavior and execute some other code we simply must overflow a buffer and rewrite the return address. But where do we write the code to be executed? In the buffer itself of course! For this to work we must make sure the program is able to execute commands that are found on the stack.

To make the stack be able to execute commands:
* $ gcc -z execstack

Furthermore, we should disable the compilers’ stack guard protection.

To turn off stack guard protection:
* $ gcc -fno-stack-protector

The trickiest part of this attack is finding out where the return address is that we need to overwrite. This is especially complicated with address space randomization because it means we will need to figure out the location of the return address on the stack each time we wish to use this attack on this program. Turning it off still leaves us with the question of the exact location we need to overwrite- and even more critically with what.

To turn off address space randomization:
* $ sudo sysctl -w kernel.randomize_va_space=0

One can attempt to brute force guesses until you get the return address overwritten, but for simplicity we will find out the location of the return address with gdb. A neat trick to get the return address to point to where you want it to is to fill your shellcode with NOP for a huge range before your actual code to run. When the instruction pointer encounters a **NOP** it will move on to the next instruction. In short, you don't have to return to the exact start of the code you wish to execute, just somewhere below the starting point!

The shellcode given in the SEED Labs invokes the execve() system call to execute /bin/sh which is symbolically linked to a shell process. Another issue is that because we are copying into the buffer with strcpy(), nowhere in our code can there be a byte of all 0s. This will be seen as a null char and stop the buffer copy.

<code> // This shows an example overwriting of the buffer. Bytes 36-39 is the overwriting of the return address.

    buffer[36] = 0xDC;
    buffer[37] = 0xEB;
    buffer[38] = 0xFF;
    buffer[39] = 0xBF;
    int i;
    int j = 0;
    int len = 512 - (sizeof(shellcode) / sizeof(shellcode[0])); // 512- buffer length
    printf("%d", len);
    for(i = len; i <= 512; i++){
        buffer[i] = shellcode[j];
        j++;
    }

Finally, I ran the code again but this time with address randomization turned on. Brute force was used to guess the correct location of the return address.

![alt]({{ site.url }}/images/posts/buffer_overflow.jpg)
