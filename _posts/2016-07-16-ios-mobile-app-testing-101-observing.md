---
layout: post
title: 'iOS Mobile App Testing 101: Intercepting and Observing Mobile Application
  Traffic'
date: '2016-07-16T16:05:00.001-06:00'
author: Sanjiv Kawa
tags:
- iOS
- Proxy
- Mobile Application Testing
- Burp
- Penetration Testing
comments: true
---
Establishing a Man-in-the-Middle (MitM) position can give you a great deal of insight into the various HTTP requests and API calls an iOS mobile application makes.

Being able to intercept and observe application traffic can be achieved by:

* Configuring Burp to listen on an interface accessible from the local network.
* Installing Burp's root CA on the iOS device. This is so that accessing SSL/TLS protected applications is seamless (no verification errors).

<figure>
  <center>
    <img src="https://4.bp.blogspot.com/-EVhb3AhykbA/V4qlUeeDyWI/AAAAAAAACEw/MNQyFZaXFXgL2PmaapdzM6xC7QEcycZfQCEw/s1600/1.png">
    <figurecaption>Configure Burp to listen on an interface accessible from the local network.</figurecaption>
  </center>
</figure>
<br><br>
<figure>
  <center>
    <img src="https://3.bp.blogspot.com/-0yD_Q4eUWi0/V4qpmzKxw-I/AAAAAAAACE4/aJT4yW-7E_IuOavB6wutB5qginqsqJK1QCLcB/s1600/0.png">
    <figurecaption>Open your browser of choice and point it at Burp.</figurecaption>
  </center>
</figure>
<br><br>
<figure>
  <center>
    <img src="https://2.bp.blogspot.com/-Nzh7m6yEhqk/V4qlUnXL9MI/AAAAAAAACEw/6-ZPma-DJ0g9ZJ7CFtglJoZ04-fvcoDxACEw/s1600/2.png">
    <figurecaption>Ensure that the certificate is verified by "PortSwigger"</figurecaption>
  </center>
</figure>
<br><br>
<figure>
  <center>
    <img src="https://1.bp.blogspot.com/-Qi7CbpJM0q0/V4qlU3okBSI/AAAAAAAACEw/TrJaVViySx8gAtYugh8szRggAnaiENEBQCEw/s1600/3.png">
    <figurecaption>Select the PortSwigger root CA and export it.</figurecaption>
  </center>
</figure>
<br><br>
<figure>
  <center>
    <img src="https://4.bp.blogspot.com/-VVMvXVmzR3g/V4qlU5BJVpI/AAAAAAAACEw/m6SaeIfW9AodG9lKWAI59MCq1h_Ah_SYwCEw/s1600/4.png">
    <figurecaption>Make sure that the certificate is saved as a .CER file, but exported as a X.509 DER certificate.</figurecaption>
  </center>
</figure>
<br><br>
<figure>
  <center>
    <img src="https://3.bp.blogspot.com/-7tjQwwUFE6U/V4qqjPeAzEI/AAAAAAAACFA/DvzSFgmpNj4YShcz0o6bdy0JTQ3wO_NFQCLcB/s1600/1.png">
    <figurecaption>Spin up a temporary web server using Python's SimpleHTTPServer module.</figurecaption>
  </center>
</figure>
<br><br>
<figure>
  <center>
    <img src="https://4.bp.blogspot.com/-fJKdgPdJB7M/V4qsKO-gR2I/AAAAAAAACFM/zMMYPiiej4oRBhAQSZufxkj7RqBqhntwwCEw/s1600/6.png">
    <figurecaption>Use Safari on your iOS device to navigate to http://ip:10000 and click on PortSwiggerCA.cer</figurecaption>
  </center>
</figure>
<br><br>
<figure>
  <center>
    <img src="https://1.bp.blogspot.com/-sGIJJYiD-lg/V4qlVO8iuqI/AAAAAAAACEw/xC2MQR-9B0cIn6_hIT0xwRY597m8Ik-ywCKgB/s1600/7.png">
    <figurecaption>Install Burp's root CA certificate.</figurecaption>
  </center>
</figure>
<br><br>
<figure>
  <center>
    <img src="https://1.bp.blogspot.com/-1mE8PcTBDoY/V4qlVDhQUmI/AAAAAAAACEw/2P5hAtUTyMgTuiMPJ40XpqtANhh2Gxc3ACKgB/s1600/8.png">
    <figurecaption>Ensure that the configured profile is "PortSwigger CA".</figurecaption>
  </center>
</figure>
<br><br>
<figure>
  <center>
    <img src="https://2.bp.blogspot.com/-RsGU2QNDqQc/V4qlVTHFrUI/AAAAAAAACEw/n3gHNDUNOiMNBkAVC3AeleADpHvrNLgLQCKgB/s1600/9.PNG">
    <figurecaption>Navigate to network settings and point the iOS wireless interface to Burp's proxy.</figurecaption>
  </center>
</figure>

That should wrap up all of the requisite configurations needed to observe native iOS application traffic as well as browser traffic.

Here is an example of capturing authentication details from the iOS Mail application:
<figure>
  <center>
    <img src="https://4.bp.blogspot.com/-EkzcvDjiZRc/V4qvJFtF0nI/AAAAAAAACFY/K8MpRsFcjlItGClSqwDtwKE3swfUy73rgCLcB/s1600/Untitled.png">
    <figurecaption>Refreshing the Mail application.</figurecaption>
  </center>
</figure>
<br><br>
<figure>
  <center>
    <img src="https://4.bp.blogspot.com/-rfJ_rXPbunM/V4qlUYWGtaI/AAAAAAAACEw/_HGcF2Ra_wc4ws9JtO4rgVz8AkUhKS0vACKgB/s1600/12.png">
    <figurecaption>Intercepting the Basic Authentication request to Microsoft's mail servers.</figurecaption>
  </center>
</figure>
<br><br>
<figure>
  <center>
    <img src="https://1.bp.blogspot.com/-oyg-MFVDI8c/V4qlUleHmII/AAAAAAAACEw/e80NSiHmORY66psPYd4nx0IUP5wY_PRTwCKgB/s1600/13.png">
    <figurecaption>Decoding the captured Base64 encoded authentication details.</figurecaption>
  </center>
</figure>
