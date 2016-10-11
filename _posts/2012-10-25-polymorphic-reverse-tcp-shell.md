---
layout: post
title: Polymorphic Reverse TCP Shell
date: '2012-10-25T16:44:00.000-06:00'
author: Sanjiv Kawa
tags:
- Information Security
- shikata_ga_nai
- Metasploit Framework
- Polymorphic Malware
- Reverse Shell
- exe2vba.rb
- Backtrack R5
- Penetration Testing
- Microsoft Office
- Windows 7
- Research
- Video
comments: true
---
<iframe width="560" height="315" src="//www.youtube.com/embed/91p7wzQ0CaE" frameborder="0"> </iframe>

I created a polymorphic reverse shell in the MSF which I then embedded into a word document through the form of a VBScript + word macro. Essentially this split up body of the payload so that the virus decryption routine was nested within a macro and the encrypted virus body just sat in the body of a word document. Upon execution of the word document the macro will decrypt the the encrypted virus body, thus resulting in a reverse shell successfully being accessed on the attackers machine.

This attack vector could easily be mitigated by forcing a GPO to restrict macros from being launched.

Hopefully you find this as exciting as I did.

Thanks for reading.
