---
layout: post
title: Understanding Rootkits
date: '2012-12-03T02:01:00.000-07:00'
author: Sanjiv Kawa
tags:
- Academia
- Information Security
- FU_Rootkit
- Malware
- Rootkits
- Research
- Information Technology
comments: true
---
Sorry about the lack of updates, I have been enjoying my time off after being approved for graduation.

I thought I would upload some research that I have done on Rootkits, hopefully this can help other people conducting research on this area of study.

The content of this report is based on a meticulous dissection with regards to the internal workings of rootkits. Over the course of this document various aspects of the malicious software will be outlined and then deeply scrutinized. Numerous underlying key concepts will be brought to light in order to apprehend this structured document.

Enjoy,

Sanj.

# 1. Introduction

Rootkits are a sinister presence within the IT security world. In December of 2007 PC World issued an article stating that 20% of networked computers were infected with a type of rootkit [1]. This is a troubling figure and although security protocols have tremendously evolved since 2007, so has malware. The struggle to contain and counteract malware is a persistent cat and mouse game where the white hat security corporations seem to be chasing the ever-growing community of black hat hackers. Within this report, rootkits will be explored so that an understanding can be derived of how powerful, harmful, and most importantly how silent a rootkit is on a victims system.

# 2. Analysis

Within the analysis section of this report there will be five main theoretical fields of focus whilst researching Rootkits. The following areas are as follows:

2.1 Exploring Rootkits and their workings
2.2 Classifications and Types of Rootkits
2.3 The Installation Process of Rootkits
2.4 Rootkits Concealing their Existence
2.5 Detection, Removal and Countermeasures

## 2.1 Exploring Rootkits and their workings

Rootkits are often a tricky concept for the average user to grasp. This is mostly due to the complexity and operation of the malware. In order to become acquainted with the technical discussion that will take place within this report, a few terms must first be outlined.

### 2.1.1 What is a Rootkit?

If the expression “rootkit” is broken down, an extrapolation can be made on the two derived terms; root and kit. Focusing on the term root:

An operating system has implementations in place to control the access level that a user has. For example, if a Windows user is using the “Guest Account” they are restricted from resources in certain ways. Namely, executing programs for installation, configuration restrictions, and saving data.

By observing these restrictions an understanding can be made. Such that while a user is within the “Guest Account” a user cannot achieve much progress in terms of modification to the operating system. This is due to the fact that the “Guest Account” is at a lower hierarchical access privilege than the “Administrator Account”.

The “Administrator Account” in UNIX/Linux systems is called root, as it is the highest hierarchical access privilege one can attain. Both terms provide the same meaning, what this infers to is complete and total control of a system. Including control over network traffic, overall system monitoring, and access to absolutely every configuration file available.

Now that some helpful concepts have been outlined, the question that remains is, what is a Rootkit?

Well, simply put, a rootkit installs unauthorized and undesired collections of programs in order to gain and maintain root access on a system whilst hiding all evidence of their existence. Rootkits are derivation of malicious software, categorized as malware.

### 2.1.2 Illustrating the Purpose of Rootkits

Rootkit’s are often referred to as stealthy and highly malicious. In most cases the user of a system will not realize that a rootkit has been installed. The reason why this type of malware is so advanced in concealing itself from the user is due to the way it hides its existence. This is achieved by modifying certain files on a system; in the case of Windows this includes the registry, various program files and dynamic link library files (.dll) [2].

Before this report transitions into the explanation of how rootkits conceal themselves, it is useful to define the classification and types of rootkits.

## 2.2 Classifications and Types of Rootkits

Often a classification can be judged by whether or not a rootkit can survive a system reboot. This brings in the concepts of volatile memory and non-volatile memory.

Volatile Memory is memory that is considered to allocate size and allow data to be stored temporarily. This data is then erased on system restart or system shut down. For the purposes of contrasting this to rootkits, volatile memory can be represented as RAM.

Non-Volatile memory on the other hand is considered to be memory that allows data to be stored persistently. Therefore after a system has restarted or after a system has shut down the data still exists. This is usually represented as a hard disk or solid-state drive.

Based on volatile and non-volatile memory, rootkits can be broken down into several distinguishing types.

* Persistent: Persistent rootkits are activated each time the system boots, this requires the malicious code to be stored in persistent memory, such as the registry or file system; located within the operating system on the hard drive. No user interaction is required in order for this rootkit to execute [2][3].
* Memory Based: This is a "temporary" rootkit as it is stored in volatile memory (RAM). It has no way of surviving after a reboot is performed [2][3].
* User-Mode: User-Mode rootkits sit as a middleman in between the users calls and the operating systems application program interface (API). The way this works is that the users calls are intercepted and the returned result is modified. An example of this is if the user wants to view a certain directory, the return result would not include any files or entries that are associated with the rootkit [2][3].
* Kernel-Mode: Kernel-Mode rookits work similarly to user-mode rookits, however they intercept calls that are being made to the Kernel. The rootkit can also hide malicious software by taking it off the kernels running task list [2][3].

User mode rootkits run at security Ring 3 whereas kernel mode rootkits work at security Ring 0 level. The kernel mode rootkits is extremely powerful as well as the most advanced. The complexity behind the latter rootkit type allows it to effectively subvert the kernel.  

It is also useful to note the taxonomy breakdown of a rootkit [4]:

* Rootkits are not self-replicating.
* Rootkits themselves have no population growth.
* Rootkits are parasitic; they need a host to survive.

## 2.3 The Installation Process of Rootkits

Various types of malware require a weakness in the operating system to be effective; this is formally noted as a vulnerability. Malware will then penetrate/exploit this vulnerability in order to gain access to the system and execute its malicious code. Certain malwares will also drop a payload.

One way for rootkits to find their way on a victim’s computer is to be attached as a payload to a worm, such as a Trojan Horse. To further clarify this, the worm will crawl the network in search for a system that has vulnerabilities.
Once the worm finds a node that has a particular vulnerability, it will exploit the hole and gain access to the system. From there, the worm will execute its malicious code and then drop the payload, which in this case is a rootkit [2].
Alternatively another way that rootkits are installed on a system is from the direct activity from a hacker exploiting a system. This is achieved in the following steps:

1. The hacker runs a network scan that will identify open ports on one system or a range of systems. This is achieved using a utility called nmap [2][5].
2. The hacker exploits the port via the correlating process that utilizes the open port and gains access to the system [2].
3. Once within the system, the hacker can use various techniques to escalate the current privilege of the system to root. These attack techniques include brute force password cracking, as well as utilizing various other pieces of malware [2].
4. The attacker places the rootkit on the victims system. The rootkit may also contain a payload such as DOS bot, or a backdoor. The rootkit is then executed [2].

## 2.4 Rootkits Concealing their Existence

The incredibly impressive yet terrifying aspect of a rootkit is how they hide their existence on a victim’s computer. In order to understand the method in which rootkits conceal their existence, first a familiarity must be drawn with processes and process tables.

### 2.4.1 Processes, Process Tables and Scheduling

When the user or system executes a program an instantiated process is spawned.  There are many processes that concurrently run on the system, each of which are assigned a unique process ID. Processes are all allocated a sufficient amount of resource and time to run based on many factors including priority. The scope of this paper does not require the in depth explanation of processes, this basic definition should fulfill the fundamental understanding [6][7].

Process tables are a core component of the Linux/UNIX Kernel, their main task is to manage, schedule and keep track of processes. A primitive ideology that can be constructed of the process table is that it is a large list-style compilation of all the processes running on the system. One of the categorization columns of the process table is the PID (process ID) [7].

The scheduling feature of the aforementioned process table allows processes to be allotted a specific amount of time while accessing certain resources, such as memory or the CPU. There are many functionalities of the task scheduler, including various queues. Theses queues represent the current state of the process, two of which are whether the process is ready to run or currently running.

The FU rootkit hides its process by taking advantage of the Windows process table. It extracts the PID of a currently running process and replicates it. The process that is currently using the assigned PID is not taken off the running queue and is unaffected in terms of functionality. FU is a kernel mode rootkit; further discussion will take place in 2.4.3.

### 2.4.2 User Mode Rootkits

User mode rootkits operate amongst the same security ring as user programs. As stated in section 2.3, there are a numerous possibility of installation routes. The goal of the user mode route kit is to completely manage and control the calls coming though and from the API [8].

A common attack method for user mode rootkits is to practice a method called dynamic link library (dll) hooking. This effectively allows the malware to execute its malicious code within a process; the purpose of this is to spoof the actual process [8][9].

Typically a user mode rootkit sits as a middleman in between the users calls and the operating systems API. This permits the rootkit to intercept events in order for it to cloak the manipulations it is making to the operating system from the victim. Consider the following scenario:

A Windows operating system has various applications installed on it, some of which are third party. A user mode root kit will search for a dll, considerable amounts of which are undocumented. Once a dll is found, the structure and nature of this shared library schema allows the rootkit to hook into the vulnerable dll and have access to all other dynamic link libraries. Within this the rootkit is able to execute it’s malicious code. Thus allowing the hacker to have complete hidden control [8][9].

Other than dll hooking, another method of practice for rootkits is to vigorously take over the memory allocation of the process by erasing and rewriting it with their own. This is a “dirtier” alternative, as it will cause the application to crash, thus exposing a problem to the victim [8][9].

### 2.4.3 Kernel Mode Rootkits

Kernel mode rootkits run at root level, as previously mentioned (2.1.1) this is highest hierarchical access privilege one can attain. This type of rookit works similarly to user-mode rookits, however they intercept calls that are being made to the kernel using a concept called DKOM (Direct Kernel Object Manipulation).

The interception process becomes effective, as some privileged application level programs will need to access the kernel to carry out their tasks. A valid reason as to why an application would need to touch the kernel could be to access a driver [10].

To elaborate on DKOM the methodology behind the concept is quite simple.
When a privileged application requires access to the kernel, an object is created. A kernel mode rootkit will modify the contents of the object without triggering the operating system to believe that the object has been modified. This is theoretically forgery of the object, however the operating system will not realize the attack and it will carry out its duties as programmed without alerting the user [10].

The forgery process is largely due to the amount of unsigned drivers that exist on a Windows system. A countermeasure has only recently been taken to only allow signed drivers to be available to the application layer for object access. Moreover, the Stuxnet rootkit was able to acquire a verified driver and integrate it into its rootkit. The horrendous aspect of this revolves around one of the signed drivers that were integrated. It is manufactured and verified by a company called Realtek, which is responsible for a large majority of sound cards on modern systems [11].

Kernel mode rootkits do not need to use the hooking method as they have already subverted the application layer with direct object manipulation into the kernel. Due to this, the rootkit will now reside within security Ring 0; which translates to a complete breach of security and an untrusted system [12].

This results in a complete lack of privacy, as the hacker is now able to control all aspects of the system, as well as use it towards any means they may have.

To this day, one of the most effective methods to removing a kernel mode rootkit is to reformat the hard drive. However this results in the user having the back up all files they may need, some of which may contain traces of either a user mode or kernel mode rootkit [2][9].

## 2.5 Detection, Removal and Countermeasures

Rootkits are potentially the most advanced style of malware that the computer security world has witnessed. Naturally this makes them incredibly difficult to detect. Kernel mode rootkits that reside at a low level of the system are much harder to identify than user mode rootkits that reside at the application level [2].

There are several ways to countermeasure a rootkit:

One effective pre-emptive measure is to use an intrusion detection system to monitor the network for the code signature of distinguishable rootkits. This IDS may have to interact with a database that stays up to date with malware signatures in order to be effective against new rootkits [2].

Another adequate means of detection would be to implement a driver based key logger that would write standard input from keyboard into a log file. If discrepancies are found between actual usage and the log file, then the system could be infected with a rootkit [2].

A downside of being infected by a user mode rootkit is that the anti-rootkit utility that is being used to counteract the malware could possibly become infected via targeted dll hooking.

If a user suspects that a kernel mode root kit has compromised their system, the only assertive way to eliminate it is by installing a fresh copy of the operating system [2][9].

# 3. Conclusion

Computer Security has been revolutionized since the introduction of large-scale public networking. An overview of this document provides the pieces to conclude a synopsis; this is that rootkits are the silent malware that violates the highest degree of personal privacy. Rootkits are able to manipulate an operating systems architecture in such an advanced and intelligent way so that they are hidden from the users eye. The installation vectors are solely based on system vulnerabilities that have not been patched, either due to lack of updating or potentially undiscovered system flaws. Even so, the countermeasures to combat rootkits still seem ineffective, especially for the average user that cannot conceive the concepts of advanced measures such as IDS or key logging. This has concealed the idea that we live in a society that is uneducated about the vast dangers of malware. Many flock to anti viruses to protect their systems, however kernel mode root kits subvert these utilities. This raises the question of how file systems will become more intelligent in the future whilst handling kernel object creations or even system calls? Given the nature of this question, we can only speculate in the present and hope that computational security evolves to the state where rootkits will be deemed legacy.

4. References

[1] Egan, M. (2007). One in Five PCs Infected With Rootkits. Web Site: http://www.pcworld.com/article/140538/one_in_five_pcs_infected_with_rootkits.html (Accessed: May 3rd 2012).

[2] Stallings, W &amp; Brown, L. (2008). Computer Security Principles and Practice. Pages 242-245. Hard Copy Text Book. (Accessed: May 3rd 2012).

[3] Romano, C. (2011). How to Remove a Rootkit from a Windows System. Web Site: http://www.technibble.com/how-to-remove-a-rootkit-from-a-windows-system/ (Accessed: May 12th 2012).

[4] Aycock, J. (2006). Computer Viruses and Malware. (Accessed: May 12th 2012).

[5] Lyon, G. (1997). Nmap. Website: http://insecure.org/fyodor/ (Accessed: May 12th 2012).

[6] Matloff, N. (2005). Unix Processes. Web Site: http://http://heather.cs.ucdavis.edu/~matloff/UnixAndC/Unix/ (Accessed: May 12th 2012).

[7] Dewan, P. (2004). Process Table. Web Site: http://www.cs.unc.edu/~dewan/242/s07/notes/pm/node3.html (Accessed: May 12th 2012).

[8] Kapoor, A &amp; Sallam, A. (2007). Rootkits Part 2: A Technical Primer. Web Site: https://www.info-point-security.com/open_downloads/alt/McAfee_Whitepaper_rootkits_part_II.pdf (Accessed: May 12th 2012).

[9] Kawa, S. (2011). ITSC 321 Journal. (Accessed: May 12th 2012).

[10] Butler J &amp; Hoglund G. (2004) VICE! Catch the Hookers. Web Site: http://www.blackhat.com/presentations/bh-usa-04/bh-us-04-butler/bh-us-04-butler.pdf (Accessed: May 14th 2012).

[11] Raiu, C. (2010). Stuxnet signed certificates FAQ. Web Site: http://www.securelist.com/en/blog/2236/Stuxnet_signed_certificates_frequently_asked_questions (Accessed: May 14th 2012).

[12] Microsoft. (2006). Driver Signing Requirements for Windows. Website: http://msdn.microsoft.com/en-us/windows/hardware/gg487317.aspx Accessed: May 14th 2012).
