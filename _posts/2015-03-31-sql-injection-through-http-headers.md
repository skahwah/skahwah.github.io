---
layout: post
title: SQL Injection through HTTP Headers
date: '2015-03-31T16:30:00.000-06:00'
author: Sanjiv Kawa
tags:
- bWAPP
- hash
- MySQL
- root
- HTTP Headers
- SQL Injection
- SQLi
- User-Agent
comments: true
---

A vulnerable web app that I have been enjoying recently is <a href="http://www.itsecgames.com/">bWAPP</a>. You should definitely check it out if you haven't already.

I came across bWAPP as I have been wanting to get better at testing injection flaws, specifically SQL, XML, SSI and general command injection. In the process of doing so, I learned about a pretty neat way to conduct SQLi attacks by tampering with the User-Agent HTTP Header.

bWAPP has a vulnerable application which logs the IP address and User-Agent of visitors into a MySQL database. As the User-Agent string is not sanitized by the application, we can manipulate the value of string and replace it with malicious SQL statements.

<figure>
  <center>
    <img src="https://2.bp.blogspot.com/-j0R1gLKMN0o/VRsdpZGHNlI/AAAAAAAABzE/Sc_xVZjSKj8/s1600/1.png">
    <figurecaption>Recording the User-Agent string and IP address of the firewall.</figurecaption>
  </center>
</figure>

<figure>
  <center>
    <img src="https://1.bp.blogspot.com/-iIcN0Rif0wg/VRsdpc4egOI/AAAAAAAAByQ/-1NWaMK1K-k/s1600/2.png">
    <figurecaption>Checking out the User-Agent string in Burp.</figurecaption>
  </center>
</figure>

<figure>
  <center>
    <img src="https://4.bp.blogspot.com/-xm22VnnOGjg/VRsdpaW-SQI/AAAAAAAAByM/Otiq8o7H2yE/s1600/3.png">
    <figurecaption>Replacing the User-Agent string with a single tick.</figurecaption>
  </center>
</figure>

<figure>
  <center>
    <img src="https://1.bp.blogspot.com/-6qzSFkAxM88/VRsdp4wNpQI/AAAAAAAAByU/JAuNN17Ykpw/s1600/4.png">
    <figurecaption>Received a pretty verbose MySQL syntax error that we can work with.</figurecaption>
  </center>
</figure>

<figure>
  <center>
    <img src="https://4.bp.blogspot.com/-H0aSlGNJAtQ/VRsdqF6LiZI/AAAAAAAAByY/1-Hd7MyAQww/s1600/5.png">
    <figurecaption>Based on the MySQL syntax error, I decided to put together a simple statement following the syntax rules. I then set some canaries to see where they were going to be presented on the front end.</figurecaption>
  </center>
</figure>

<figure>
  <center>
    <img src="https://3.bp.blogspot.com/-0uTwKhDdnyA/VRsdqZK28YI/AAAAAAAAByo/HAbfHK91dQY/s1600/6.png">
    <figurecaption>Looks like the MySQL query which replaced the User-Agent string worked!</figurecaption>
  </center>
</figure>

<figure>
  <center>
    <img src="https://2.bp.blogspot.com/-gmes7Enpyyc/VRsdqmm4xVI/AAAAAAAAByg/MTW5Xs1ixgo/s1600/7.png">
    <figurecaption>Lets try to grab the MySQL root users hash from the database.</figurecaption>
  </center>
</figure>


<figure>
  <center>
    <img src="https://1.bp.blogspot.com/-UJSzYfzfkao/VRsdrbAkflI/AAAAAAAABys/XlegzR7-0D0/s1600/8.png">
    <figurecaption>Time to fire up hashcat.</figurecaption>
  </center>
</figure>

Thanks for reading!
