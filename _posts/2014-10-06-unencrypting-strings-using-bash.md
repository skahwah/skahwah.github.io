---
layout: post
title: Unencrypting strings using bash
date: '2014-10-06T08:38:00.001-06:00'
author: Sanjiv Kawa
tags:
- ROT13
- Decryption
- base64
comments: true
---

As I mentioned in an earlier blog post, I've recently been messing around with various CTF challenges. Often, to advance to the next level you usually need to hunt for eggs which contain encrypted/encoded passwords.

After finding the password file you need to decrypt/decode the encrypted/encoded password string. Most of the time you're told what the string is encrypted in, probably out of mercy so you don't spend an afternoon banging your head against a desk.

There are some neat ways to decrypt strings through bash, here are a few ways:

If you're ever in the rare situation where you need to rotate an alphabetical set by 13 characters (Caesar Cipher) you could use the tr command.

<figure>
  <center>
    <img src="https://3.bp.blogspot.com/-Load63ogBJ0/U02qy401M8I/AAAAAAAABms/twTtGrNoKPU/s1600/Kali+x64-2014-04-15-15-55-06.png">
  </center>
</figure>

Or in the situation where something is encoded by base64 you can use the built-in base64 command.
<figure>
  <center>
    <img src="https://4.bp.blogspot.com/-s3EBkoax2Iw/U02rxmJ8IxI/AAAAAAAABm0/8zqWJWvaKFw/s1600/Kali+x62-2014-04-15-15-58-45.png">
  </center>
</figure>

There are plenty more ways to decrypt or decode strings. A neat tool that I have used to identify which hash algorithm is being used on cipher text is <a href="https://code.google.com/archive/p/hash-identifier/">Hash Identifier</a>.
