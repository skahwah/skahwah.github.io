--
layout: post
title: SLAE Assignment 3
date: '2016-12-04T20:30:00.000-06:00'
author: Sanjiv Kawa
tags:
- SLAE
- SecurityTube
- Assignment
- Shellcode
- nasm
- assembly
- egg hunter
- access syscall
comments: true
---
### SLAE Assignment Stub
This blog post has been created for completing the requirements of the SecurityTube Linux Assembly Expert certification.

<a href="http://www.securitytube-training.com/online-courses/securitytube-linux-assembly-expert/">http://www.securitytube-training.com/online-courses/securitytube-linux-assembly-expert/</a>

Student ID: SLAE-807


### Assignment 3 Requirements
- Study the egg hunter shellcode. It is similar to a two stage shellcode.
- The first stage searches for a pattern it can find in memory. That pattern signifies where the second stage lies which needs to be executed.
- Create a working demo of an egg hunter. It should be configurable for different payloads. This means that once the pattern has been found the second stage should be easily configurable.


### What are Egg Hunters?
First off, I want to thank Skape for putting together a paper titled <a href="www.hick.org/code/skape/papers/egghunt-shellcode.pdf">Safely Searching Process Virtual Address Space</a>. This really helped me understand how to write Linux x86 egg hunters that are robust, reliable and can safely traverse the virtual memory address space of an arbitrary process.

Egg hunters are incredibly useful from an exploitation perspective. There are scenario where a small amount of space is accessible in a deterministic location of an arbitrary process. This space is usually too small for shellcode, but can house an egg hunter.

Shellcode is then placed somewhere else in the same arbitrary process, however the specific location is unknown.

The purpose of an egg hunter, in terms of exploitation, can be broken in to two stages:

- Stage 1 consists of traversing the virtual memory address space for a process and inspecting all memory locations for a specific, yet arbitrarily constructed four byte value. This value is referred to as an egg, for example: `\x41\x42\x41\x42`. There are some cases where an egg needs to avoid certain bad characters or contain valid instructions, this is based on the design of the egg hunter, and sometimes restrictions imposed by the process.

- Stage 2 consists of passing execution to the memory location in the same arbitrary process that contains the shellcode. This egg is placed at the beginning of shellcode, typically twice, and signifies the start of a series of malicious instructions. For example: `\x41\x42\x41\x42\x41\x42\x41\x42[execute shell on remote system]`.

The reason why the egg is placed twice at the beginning of the shellcode is because it enables the egg hunter to search for a value in memory that has the same four byte values which are one after the other.


### The Shellcode
The example shellcode I will be using is a really simple stack-based `execve` SYSCALL that executes `/bin/sh`.

The nasm source is located <a href="https://github.com/skahwah/slae/blob/master/execve-stack/execve-stack.nasm">here</a> and a brief breakdown of the shellcode is located <a href="https://popped.io/execve-stack/">here<a>.


Here are the opcodes:

```
\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80
```

### The Egghunter
The egg hunter is responsible for searching for the shellcode in the virtual memory address space of an arbitrary process.

The egg hunter uses the `access` system call to identify valid memory addresses by means of the `EFAULT` error code. If this error code is returned by the `access` system call, the page value is simply incremented to the next page until a valid memory address is found.

Once a valid memory address is found, the `scas` instruction is used to compare the `DWORD` value in the current memory location against the `egg`. If the contents of the current memory location do not contain the `egg`, then the address is incremented and the search continues until the `egg` is found. At this point, execution of the process is directed to the memory address directly after the located `egg`, which signifies the start of the `execve` `/bin/sh` shellcode.

The egg hunter can be found <a href="https://github.com/skahwah/slae/blob/master/assignment3/egghunter.nasm">here</a>.

I have also created a "test" program written in C. This uses the egg hunter to traverse the virtual memory address space for the "test" process for a user supplied egg. The egg does not need to consist of executable instructions.

This egg is prepended to the stack-based `execve` `/bin/sh` shellcode. Once the egg is found, execution is passed to the shellcode.

The test program can be found <a href="https://github.com/skahwah/slae/blob/master/assignment3/egghunter-test.c">here</a>.

Compile it using these options: `gcc -fno-stack-protector -z execstack egghunter-test.c -o egghunter-test`


### Dissecting the Egg Hunter
1\. Clearing the direction flag and registers

The first few instructions are responsible for clearing the `EAX` and `EBX` registers. The direction flag is also cleared to prevent the egg hunter from failing as the scas instruction is used. It may be possible that this flag is set by the process, although unlikely.

```nasm
cld		
xor eax, eax`
xor edx, edx
```

<br>2\. `next_page:`

The `next_page:` procedure is responsible for aligning the page value of the current memory address. It largely serves as an incrementor to the next page size if an invalid memory location is returned by the `access` system call.

The first address of relevance is typically 0x8048000, however this egg hunter does not assume that the virtual memory address space for the given process follows conventional means. As such, the first address that is examined is `0x0001000`.

```nasm
next_page:
  or dx, 0xfff ;page alignment
```

<br>2\. `increment_address:`

The `increment_address:` procedure is largely responsible for incrementing the current address that is in `EDX` and later in `EDI` when inspected by `scas` by `1`. The purpose of this is to increment the current address in the event that the user supplied egg as not been found.

```nasm
inc edx		
```

<br>3\. The `access` system call

The `access` system call, `0x21` or `33`, simply takes the value currently in the `EBX` register, which represents a memory location, and attempts to access it. The return value of the `access` syscall is stored in `AL` and indicates if the supplied memory location is valid.

The `EBX` register contains the value in the `EDX` register plus 4. This is so that eight contiguous bytes of memory can be searched.

The `EBX` register contains the the only argument required by the `access` system call, a memory location.

```nasm
lea ebx, [edx +0x4]
mov eax, 0x21		
int 0x80
```

<br>4\. `search_vas:`

The `search_vas:` procedure is largely responsible for comparing the `DWORD` value in the current memory location against the `egg` and directing execution of the process to shellcode once the `egg` has been located. However, this can be dissected more granularly and other interesting elements in this procedure can be touched on too.

Firstly, a comparison is conducted against the value currently in the `AL` register and `0xf2`.

`cmp al, 0xf2`

The return value of the `access` system call is stored in `AL` and indicates if the supplied memory location is valid. If the memory location is not valid, `0xf2`, which represents the low byte of the `EFAULT` return value is stored in the `AL` register.

If `0xf2` exists in `AL` the current memory location is invalid. The zero flag `ZF` is set, as comparing `0xf2` by itself is `0`. This triggers the next instruction `je next_page`	to execute, which jumps to the `next_page` procedure. As mentioned in Step. 2, this procedure increments to the next page value.

If `AL` does not contain `0xf2`, then a valid memory address has been accessed by the `access` system call and the previous instruction in skipped as the zero flag `ZF` will not be set. The following instruction simply moves the `egg` in to the `EAX` register.

`mov eax, 0x534B534B ;SKSK` 	

The reason for this is that the `EAX` register is one of the native IA-32 instructions for doing string based comparisons with `scas`. In other words, the value in the memory location currently inspected by `scas` are compared against the value in `EAX`.

The next instruction takes the value in `EDX`, which is a memory address, and places it into `EDI` as `EDI` is used by `scas` when inspecting the contents of a memory location.

In addition, the current memory address in `EDX`, and now `EDI` is currently four bytes behind the value in `EBX`, which was the memory address accessed by the `access` system call.

`mov edi, edx`

The next instruction takes the `DWORD` stored in `EAX`, which is the `egg` `0x534B534B` and compares it against the value in the memory location referred to be the `EDI` register.

`scasd`

If the value in the memory location pointed to the `EDI` register does not match the value in the `EAX` register, essentially meaning that the egg has not been found, then the zero flag `ZF` will not be set. This triggers the next instruction `jne increment_address`	to execute, which jumps to the `increment_address` procedure. As mentioned in Step. 1, this procedure increments the value in `EDX`, which is later stored in `EDI` to the next memory address to be searched by `scasd`.

Otherwise, if the zero flag `ZF` is set, this indicates that the `egg` has been located in the memory location currently pointed to by `EDI`. As comparing the `egg` in `EAX` with the value in `EDI`, which is also the `egg` results in a `0`.

`scasd` then increments the value in `EDI` by four bytes. This effectively shifts the comparison done by `scasd` to the next memory address after the `egg`. This is the memory location for the duplicated `egg` before the beginning of the shellcode.

`scasd` will once again set the zero flag `ZF` which indicates that the `egg` has been located in the memory location currently pointed to by `EDI`. As comparing the `egg` in `EAX` with the value in `EDI`, which is the second `egg` results in a `0`.

`scasd` then increments the value in `EDI` by four bytes. The address in `EDI` now contains the memory location pointing to the beginning of the shellcode after the eight `egg` bytes.

The zero flag `ZF` is set, once again, thus bypassing the next instruction which is `jne increment_address` and executing the following instruction `jmp edi`.

`EDI` currently contains the memory address to where the second stage exists, this is the address contains the beginning of the shellcode after the eight `egg` bytes.

As such, once `jmp edi` is executed, the process performs an unconditional jump to the memory address where the second stage exists and passes control to the shellcode.

```nasm
cmp al, 0xf2			
je next_page			
mov eax, 0x534B534B
mov edi, edx
scasd
jne increment_address		
scasd				
jne increment_address		
jmp edi
```
