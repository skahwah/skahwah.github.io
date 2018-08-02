---
layout: post
title: 'Incident Response 101: Preserving Information'
date: '2014-10-31T10:28:00.000-06:00'
author: Sanjiv Kawa
tags:
- Incident Response
- Preserving Information
comments: true
---
Often most people's immediate response to a malware infection is to disconnect a host from the network. While this is a great precautionary action to prevent malware propagation, it destroys some valuable data that can be used for further investigations. This includes:

* Active network connections
* Active processes which rely on active network connections

Scenario: Lets assume, that an attacker has a reverse shell on a host in your environment. If we disconnect the host from the network, that established network connection dies. Therefore, you have no way of identifying who the attacker is, what their IP address is, and you have no way to prevent further communications by means of blocking the attackers IP address at the firewall level.

As a result, disconnecting an infected host is not the first thing that should be done when responding to an incident. Information gathering and the preservation of that information is.

The information that should be gathered consists of time stamps, network connections which are listening and/or established, system information, current running processes and users in the administrators group.

I've created a pretty speedy script (<5 seconds run time) which gathers all of this information in a text file named $hostname.txt on the Desktop of the current user. The script can be found <a href="https://github.com/skahwah/skahwah.github.io/raw/master/_data/IR_DUMP.zip">here</a>.

After this information is gathered, the next steps in the incident response procedure can be taken. This is where you can determine whether network isolation is necessary or not.

Here is a sample of the output generated for my script:
{% highlight css %}
[*] IR DUMP  
 [*] Created by: Sanjiv Kawa  
 [*] Starting IR DUMP  
 [*] Captured on Fri 10/31/2014 10:15:10.06  
 [*] Finding ESTABLISHED TCP Connections  
  TCP  192.168.1.2:64817  132.245.81.130:443   ESTABLISHED   2600  
 [*] Finding TCP Services LISTENING for Connection  
  TCP  0.0.0.0:80       0.0.0.0:0       LISTENING    5824  
  TCP  0.0.0.0:135      0.0.0.0:0       LISTENING    344  
  TCP  0.0.0.0:443      0.0.0.0:0       LISTENING    2584  
  TCP  0.0.0.0:445      0.0.0.0:0       LISTENING    4  
 [*] Finding ESTABLISHED UDP Connections  
 [*] Finding UDP Services LISTENING for Connection  
 [*] Getting Running Processes  
 avp.exe            2600 Services          0   66,640 K  
 Skype.exe           5824 Console          1  165,424 K  
 [*] Getting System Information  
 OS Manufacturer:      Microsoft Corporation  
 System Type:        x64-based PC  
 Windows Directory:     C:\windows  
 System Directory:     C:\windows\system32  
 Hotfix(s):         92 Hotfix(s) Installed.  
               [01]: KB2899189_Microsoft-Windows-CameraCodec-Package  
 [*] Getting Local Administrator Accounts  
 Alias name   administrators  
 Comment    Administrators have complete and unrestricted access to the computer/domain  
 Members  
 -------------------------------------------------------------------------------  
 Administrator  
 sanjiv  
 The command completed successfully.
{% endhighlight %}
