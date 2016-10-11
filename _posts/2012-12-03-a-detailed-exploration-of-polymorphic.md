---
layout: post
title: A Detailed Exploration of Polymorphic Malwares
date: '2012-12-03T01:58:00.000-07:00'
author: Sanjiv Kawa
tags:
- Academia
- Information Security
- Security
- Obfuscation
- Decryption
- Malware
- Research
- Information Technology
- Encryption
- Polymorphic Malware
comments: true
---
Sorry about the lack of updates, I have been enjoying my time off after being approved for graduation.

I thought I would upload some research that I have done on Polymorphic Malwares, hopefully this can help other people conducting research on this area of study.

Enjoy,

Sanj.


# 1. Introduction

Information security is continually advancing as the achievements in technology progress. Traditionally the White Hat security specialists have been thwarting Black Hat cyber criminals by strategically staying one step ahead of the frontier. However, cyber criminals are finding intelligent ways of exploiting vulnerable applications in order to gain access to systems or sensitive data. A vector that is adopted by the Black Hat community is the use of Polymorphic Malware; a sophisticated trend that is on the rise.

In July of 2011 Symantec reported that 23.7% of intercepted emails contained malware samples that were “aggressively unstable or rapidly changing forms of generic polymorphic malware.” A disturbing certitude is that polymorphic malwares are incredibly efficient at bypassing traditional anti-malware detection systems, and the samples discovered by Symantec were designed for that very purpose [1].

# 2. Analysis

Within the analysis section of this report there will be five main theoretical fields of focus whilst researching Polymorphic Malwares. The following areas are:

2.1 Exploring Polymorphic Malware and their Workings
2.2 Obfuscation, Encryption and Decryption  
2.3 Classification of Polymorphic Malware      
2.4 The Installation Process of Polymorphic Malware
2.5 Detection, Countermeasures and Removal

## 2.1 Exploring Polymorphic Malware and their Workings

Before diving into the many components that encapsulate polymorphic malware, it is ideal to become acquainted with what exactly polymorphic malwares consists of.

### 2.1.1 What are Polymorphic Malwares?

Polymorphic Malwares are just as destructive and intrusive as Worms, Trojans, or other Viruses.

A key separation between a regular worm and a polymorphic worm is that a form of polymorphic malware is much more advanced than its regular malware ancestor due to its ability to self-evolve. This process is attributed to the malware being able to mutate its malicious code in order to evade detection from most anti-malware programs.

To provide some clarity on the dissection of polymorphic malwares, I am going to introduce a simplistic scenario and then expand on the polymorphic changes.

Let us assume that there is a network with multiple nodes attached to it as end devices.  

A polymorphic spyware program is introduced to one node, its purpose is to observe and monitor all user behavior on that system. After the malware has run its investigation and flagged certain behaviors, it will then provide the information gathered back to the attacker and move to the next node on the network.

The polymorphic spyware goes through a mutation process before a new instance or iteration of the program is created. This step occurs whenever the malware is replicated [2].

Due to the mutation process, the spyware is self-evolving as the malware is able to achieve a new instance of its self. The variant copy is unrelated to its predecessor in terms of the codes structure, however it is able to carry out all of the previous exploits as the original spyware [2].

Essentially, what has been achieved is a completely new malware with the same functionality as the previous malware, but without any relations that can link the variant copy to its predecessor as it propagates through the network.

### 2.1.2 The Structure of Polymorphic Malwares

A primitive virus’ code structure will stay the same throughout all copies of the malware. Thus, the virus’s signature is the same throughout all replications and is easily identifiable by an anti-virus detection system.

To create some obfuscation, early virus developers decided to improve the way that signatures were generated in order to evade detection. This came in form of encrypting the virus [3][4].

An encrypted virus consists of two parts, the first is the encrypted virus body (EVB), and the second is the virus decryption routine (VDR). When the encrypted virus is launched, the virus decryption routine will decrypt the EVB; this ensures that the encrypted virus body returns to its original form, which is a set of decrypted rules or instructions. The virus body will then execute its malicious code and potentially drop a payload, if it exists [3][4].

Before the virus has propagated to the next node or program it will make a duplicate copy of the decrypted virus body and encrypt it using a key that is randomly generated. Due to the encryption key varying on each iteration, this causes the re-encrypted virus body to change so that the succeeding version of the malware to be incomparable to the preceding version [3][4].
Figure 1 shows a graphical form of how a virus encryption works.

<figure>
  <center>
  <img src="https://3.bp.blogspot.com/-yhoVZjC0TAM/ULxnqHOqNoI/AAAAAAAAAKk/Hd_4panrLNI/s1600/1.png">
  <figcaption>Figure 1: Encryption and Decryption of the VDR and EVB.</figcaption>
  </center>
</figure>

An anti-virus detection system will find it incredibly problematic to search and extract a virus signature from the continuously changing EVB. However, White Hat specialists noticed that the VDR as this would remain consistent throughout all generations of the virus. This is when anti-virus detection systems evolved from just signature searches to byte sequence searches that could identify patterns of virus decryption routines [3][4].

As a rebuttal, malware developers were able to overcome this obstacle by creating polymorphic malwares, which are able to evade detection from most anti-malware programs through the following method:

## 2.2 Obfuscation, Encryption and Decryption

Much like an encrypted virus, polymorphic malwares contains both a virus decryption routine as well as an encrypted virus body.

Polymorphic malwares add another function within their code structure. This is known as a mutation engine, the purpose of this is to randomize the virus decryption routine after each iteration of the malicious program [4].

The polymorphic malware will have an encrypted virus body as well as an encrypted mutation engine. The decryption routine is responsible for decrypting both of these components on execution [4].

Once the malicious program has completed its set instructions, it is ready to be deployed onto another program or system. At this point the malware will replicate itself and the decrypted mutation engine into volatile memory. The malware will then call the mutation engine in order to generate a new scrambled VDR that will perform the same functions as the preceding decryption routine, however, it will contain little to no matching similarities in terms of code and structure [4].

The malware will then encrypt the virus body and mutation engine with an encryption key located in the new VDR. Once the encryption process is complete, the new VDR is appended to the recently encrypted virus body and mutation engine. This creates a succeeding version of the malware that is then added onto another program or system [4].

Polymorphic malwares improve on the previous method that encrypted virus’ used, as the decryption routine was the weak link causing detection amongst anti-malware programs. The main achievement that comes from the creation of a new decryption routine is that the VDR is now variant amongst generations of the malware [4].

When the new virus decryption routine, encrypted virus body and mutation engine are created and then encapsulated after each iteration, the variant polymorphic malware is able to evade most detections systems [4].

By creating an increasingly harder to interpret malware that can self-evolve polymorphic malware developers utilize a programming practices known as obfuscation [4].

An excellent example of this is the Stration family of malware. “Stration was changing so quickly—the encryption packaging, the compiler, everything. We saw up to 300 variants in a single day” [5].

Figure 2 shows a graphical form of how a polymorphic malware works through obfuscation, encryption and decryption.

<figure>
  <center>
  <img src="https://4.bp.blogspot.com/-305O3nFcV4U/ULxoD1RoSpI/AAAAAAAAAKs/x1Av_eryHGw/s1600/2.png">
  <figcaption>Figure 2: Obfuscation Encryption and Decryption.</figcaption>
  </center>
</figure>

## 2.3 Classification of Polymorphic Malware

Tracking polymorphic malwares based on signatures and behavior can be dependent on the speed of mutation. If a polymorphic malware takes longer to inflict damage to a system or network it is vulnerable for identification. Once it has been identified by an anti-malware detection system, the malwares signature and behavior is outputted to the anti-malwares database for analysis. Updates will then be provided based on the findings [6].

Alternatively, independent malware researchers or communities such as virustotal conduct research on polymorphic malwares and contribute their findings to a cloud-style database. This database will store information such as the malwares signature, behavior, known aliases and generations [7][8].

S, Cesare and Y, Xiang are two researchers from Central Queensland University. They have proposed a prototype classification system for polymorphic malwares. This is identified in Figure 3 [8].

<figure>
  <center>
  <img src="https://1.bp.blogspot.com/-Y1UQtG2FPDE/ULxoUHGCZGI/AAAAAAAAAK0/K7xjaSq-nSo/s1600/3.png">
  <figcaption>Figure 3: Block diagram of the malware classification system.</figcaption>
  </center>
</figure>

After analysis of this block diagram, I have determined the following:

By utilizing a honeypot, the researchers are able to capture wild polymorphic malwares. After unpacking the malware they step through its execution and generate signatures based on either a single or multiple trials [8].

A comparison is made to check if a measurable signature exists in a malware database. From there the malware is determined to be malicious or non malicious. If the malware is indeed malicious and unique then the signature is added into the database [8].

Google’s virustotal works in a fashion that is similar to this, however the community response is far greater as users are able to receive real time results after uploading a suspected malware [7].

Google utilizes a cluster of servers that run multiple instances of anti-malware detection programs in order to identify the uploaded malware. Regardless of whether the uploaded sample is malicious or not, everything is indexed and stored for future comparisons [7].

In order to test my assumptions, I decided to create a polymorphic encoded reverse TCP shell using msfencodes x86/shikata_ga_nai within the Metasploit framework. The encoding utilizes a polymorphic XOR additive feedback in order to obfuscate the payload [9].

After creating the malware, I uploaded it to virustotal, my findings are shown below in Figure 4:
<figure>
  <center>
  <img src="https://2.bp.blogspot.com/-vVXQKNDOJWI/ULxoV5tDh1I/AAAAAAAAAK8/JxitVHcidWc/s1600/4.png">
  <figcaption>Figure 4: Polymorphic malware uploaded to virustotal.</figcaption>
  </center>
</figure>

Within a minute, 42 different anti-malware programs scanned my polymorphic malware, 34 of which were able to identify it. This malware was put through a single iteration of encoding, therefore the encrypted virus body and virus decryption routine were identified fairly quickly.

## 2.4 The Installation Process of Polymorphic Malware

For a polymorphic malware to successfully install on a targeted host, it will need to bypass an anti-malware detection program. After doing so the decryption routine decrypts the virus body of the malware, which allows the body to perform its tasks in volatile memory.

The tasks within the body define the actual payload; this could be any malware that is packaged into a polymorphic form. Two examples are typical viruses and worms, both of which are explained in the following scenarios:

### 2.4.1 Polymorphic Virus

A polymorphic virus is likely to be attached to a program or executable file.

Lets assume that a polymorphic virus has been appended to an infected PDF file, which appears to be legitimate. Once the PDF is executed the polymorphic virus will also be executed, this will cause an array of potential infections to occur once the virus body is decrypted. Some rudimentary infection examples include registry edits, file tampering or propagation to other programs.

The transmission vectors that a polymorphic virus can take are vast. Some popular vectors include email, FTP, HTTP, and USB.

### 2.4.2 Polymorphic Trojan Horse

A polymorphic trojan horse can be represented as a useful program or imitate the certificate of a reputable program [10].

Lets assume that a user that is part of a network of nodes has downloaded an infected anti-virus program from an HTTP pop up window. Upon execution of the downloaded file a polymorphic trojan horse will bypass the existing anti-malware detection system and decrypt the virus body. After installation the downloaded program may represent itself as a legitimate anti-virus program and load a mock routine that appears to be scanning the users system. Meanwhile the virus body could potentially contain a reverse shell worm that will create succeeding polymorphic reverse-shell malwares to propagate throughout the network and infect other users machines.

The transmission vectors that a polymorphic trojan horse can utilize are the same as polymorphic viruses.

## 2.5 Detection, Countermeasures and Removal

Polymorphic Malwares are one of the most advanced style of malware that the computer security world has witnessed. This makes them incredibly difficult to detect due to their mutative nature.

There are several ways to detect and prevent polymorphic malwares.

### 2.5.1 Detection and Countermeasures

Pre and Post Scanning

A way of detecting a suspected polymorphic malware is by conducting a pre-scan and post-scan of the executable file. This is analyzed within a sandbox so that the executable code does not leak into crucial infrastructure.

The pre scan will create a snapshot of the codes structure. During execution a routine will step through the code in order to search for payloads or decryption routines. After execution, another analysis is constructed on the polymorphic malware to determine whether any distinguishable changes have been made to the codes structure [5].

Memory Based Detection

Polymorphic malwares will attempt to decrypt the encrypted virus body and mutate the virus decryption routine in volatile memory. A memory based signature detection program is able to determine whether or not the decrypted virus body contains a payload that is malicious due to its signature. A flaw in this procedure is that the signature must be in the detection systems database in order to be flagged as malicious. If the signature does not exist in the database then the polymorphic malware will be able to continue its execution [11][12].

Multi-Layered Approach

A multi-layered approach is the most successful way to prevent or countermeasure a polymorphic malware infecting a network or system. This would include such technologies as [10]:

* Intrusion Detection System
* Intrusion Prevention System
* System Behavior Based Detection
* Reaction to infections
* Intelligent Heuristics to compare virus vs non-virus behaviours.

To achieve a multi-layered approach, several technologies must be used throughout the networks topology. It is crucial that these technologies are synchronized and configured correctly to reduce an infection impact on the network as well as minimize false negatives and positives

### 2.5.2 Removal

Removal of a polymorphic malware is dependent on the severity of the infection and whether or not an anti-malware detection program has identified it.

For a severe case of infection Symantec has created an enterprize package called STAR Malware Protection. There is a feature within this that is related towards polymorphic malwares [13].

To trick the polymorphic malware into de-cloaking itself, the STAR Malware Protection package utilizes an advanced CPU emulation technology [13].

This is most likely used if a typical anti-malware program is unable to identify the polymorphic malware. A scenario that depicts this would consist of a polymorphic reverse-shell trying to establish communications with a master controller.  

This being said, most anti-malware detection systems are able to identify polymorphic malwares as they are mostly generated from preceding ancestors that have correlating signatures [12]. In some severe cases a user may have to reformat their system and potentially lose data that may have been compromised.

# 3. Conclusion

Polymorphic Malwares are intelligent, self-evolving programs that cause an array of intrusive and often malicious undesired effects to users. Since the creation of polymorphic malwares in 1990 [12], they have been an information security trend that is on the rise. Due to their advanced mutation and obfuscation White Hat security specialist are forced to create increasingly innovative detection methods in order to thwart the community of Black Hat malware writers. Currently, polymorphic malwares are able to exploit vulnerabilities upon a vast amount of user machines. Anti-malware agents eventually detect them, but this is often after the damage has been inflicted. Within the period between the initial infection and the point of which the malware has been found, the payload is able to execute its set instructions and potentially report sensitive data back to a master controller. In the future of information security, I believe that polymorphic malwares will still be a viable threat. Until user machines have the processing power to support efficient real time scans as well as effective prevention and detection methods to circumvent intrusion, polymorphic malwares will still be an item in an educated attackers toolbox.

# 4. References

[1] Brewster, T. (2011). Aggressive Polymorphic Malware Doubles in July. Web Site: http://www.itpro.co.uk/635194/aggressive-polymorphic-malware-doubles-in-july (Accessed: October 14th 2012).

[2] Radcliff, D. (2007). Polymorphic Malware: A Threat That Changes On The Fly. Web Site: http://www.csoonline.com/article/221190/polymorphic-malware-a-threat-that-changes-on-the-fly (Accessed: October 19th 2012).

[3] Rouse, M. (2010). Metamorphic and Polymorphic Malware. Web Site: http://searchsecurity.techtarget.com/definition/metamorphic-and-polymorphic-malware  (Accessed: October 19th 2012).

[4] Nachenberg, C. (1996). Understanding and Managing Polymorphic Viruses. Web Site: http://www.symantec.com/avcenter/reference/striker.pdf (Accessed: October 19th 2012).

[5] O’Brien, R. (2007). Polymorphic Malware: A Threat That Changes On The Fly. Web Site: http://www.csoonline.com/article/221190/polymorphic-malware-a-threat-that-changes-on-the-fly (Accessed: October 20th 2012).

[6] Smith, S.E. (2012). What is a Polymorphic Virus? Web Site: http://www.wisegeek.org/what-is-a-polymorphic-virus.htm (Accessed: October 20th 2012).

[7] Google. (2012). About virustotal. Web Site: https://www.virustotal.com/about/ (Accessed: October 20th 2012).

[8] Cesare, S. Xiang, Y. (2010). A Fast Flowgraph Based Classification System for Packed and Polymorphic Malware on the Endhost. Web Site: http://scholar.google.com.au/scholar_url?hl=en&amp;q=http://sites.google.com/site/silviocesare/academicpublications/AFastFlowgraphBasedClassificationSystemforPackedandPolymorphicMalwareontheEndhost.pdf&amp;sa=X&amp;scisig=AAGBfm2fVkqiH0Vp11ndcf3UfrlEF2688w&amp;oi=scholarr&amp;ei=uHKCUPD2MaWjigfM6YCwAg&amp;sqi=2&amp;ved=0CB0QgAMoATAA (Accessed: October 20th 2012).

[9] O’Gorman, J. (2012). Antivirus Bypass. Web Site: http://www.offensive-security.com/metasploit-unleashed/Antivirus_Bypass (Accessed: October 20th 2012).

[10] Casaretto, J. (2011). Polymorphic Malware Trends On The Rise. Web Site: http://wikibon.org/blog/polymorphic-malware-trends-on-the-rise/ (Accessed: October 21st 2012).

[11] Hosmer ,C (2008). Polymorphic and Metamorphic Malware. Web Site: http://www.blackhat.com/presentations/bh-usa-08/Hosmer/BH_US_08_Hosmer_Polymorphic_Malware.pdf (Accessed: October 21st 2012).

[12] Cluley, G. (2012). Server-side polymorphism: How mutating web malware tries to defeat anti-virus software. Web Site: http://nakedsecurity.sophos.com/2012/07/31/server-side-polymorphism-malware/ (Accessed: October 21st 2012).

[13] Symantec. (2012). STAR Malware Protection Technologies Web Site: http://www.symantec.com/theme.jsp?themeid=star&amp;tabID=2 (Accessed: October 21st 2012).

Figure 1: Created By Kawa, S. (2012), Inspired By: Nachenberg, C. (1996). Understanding and Managing Polymorphic Viruses. Web Site: http://www.symantec.com/avcenter/reference/striker.pdf (Accessed: October 19th 2012).

Figure 2: Created By Kawa, S. (2012), Inspired By: Nachenberg, C. (1996). Understanding and Managing Polymorphic Viruses. Web Site: http://www.symantec.com/avcenter/reference/striker.pdf (Accessed: October 20th 2012).

Figure 3: Block Diagram of the Malware Classification System. Created By Cesare, S. Xiang, Y. (2010). A Fast Flowgraph Based Classification System for Packed and Polymorphic Malware on the Endhost. Web Site: http://scholar.google.com.au/scholar_url?hl=en&amp;q=http://sites.google.com/site/silviocesare/academicpublications/AFastFlowgraphBasedClassificationSystemforPackedandPolymorphicMalwareontheEndhost.pdf&amp;sa=X&amp;scisig=AAGBfm2fVkqiH0Vp11ndcf3UfrlEF2688w&amp;oi=scholarr&amp;ei=uHKCUPD2MaWjigfM6YCwAg&amp;sqi=2&amp;ved=0CB0QgAMoATAA (Accessed: October 20th 2012).

Figure 4: Created By Kawa, S. (2012), Inspired By: O’Gorman, J. (2012). Antivirus Bypass. Web Site: http://www.offensive-security.com/metasploit-unleashed/Antivirus_Bypass (Accessed: October 20th 2012).
