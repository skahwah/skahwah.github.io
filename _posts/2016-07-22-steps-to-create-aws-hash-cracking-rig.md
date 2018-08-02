---
layout: post
title: Steps to create an AWS hash cracking rig
date: '2016-07-22T11:02:00.000-06:00'
author: Sanjiv Kawa
tags:
- hashcat
- amazon web services
- GPU
- cracker
- dictionary attack
- cloud
- hashes
- AWS
- crack
comments: true
---
A couple of months ago, I spun up an AWS hash cracking rig as part of a project I've been working on called Wordsmith.

After some trial and error, I thought that it would be useful to put together some provisioning steps.

I won't get into the AMI instance creation, as that should be pretty straight forward. Just make sure you select either a "g2.2xlarge" or "g2.8xlarge" instance based on your needs. I found that "g2.2xlarge" was good for me.

A quick word on cost - On average, I found that a GPU being utilized at 100% is about $1 per hour. I usually start up and power off my instance on demand; that way there are no unnecessary costs.

Connect to your AWS instance:
{% highlight css %}
ssh -i ~/.ssh/aws-key.pem ec2-user@publicIP
{% endhighlight %}

Update your box, this should take less than two minutes:

{% highlight css %}
sudo yum update
{% endhighlight %}

Next, make sure that the Nvidia GRID K520 card is recognized by the system.
{% highlight css %}
$ lspci | grep NV
00:03.0 VGA compatible controller: NVIDIA Corporation GK104GL [GRID K520] (rev a1)
{% endhighlight %}

Find and download the "recommended/certified" x64 <a href="https://www.nvidia.com/Download/Find.aspx">NVIDIA GRID K520 driver<a>:
<figure>
<center>
<img src ="https://1.bp.blogspot.com/-mJnQdcp2uuk/V5I1CGmudJI/AAAAAAAACFo/4PiKH5dW34UVOOAoNlEEwgeHhTrfh3i4QCLcB/s1600/Screen%2BShot%2B2016-07-21%2Bat%2B6.42.04%2BPM.png">
  </center>
</figure>

{% highlight css %}
$ wget http://us.download.nvidia.com/XFree86/Linux-x86_64/367.35/NVIDIA-Linux-x86_64-367.35.run
--2016-07-22 00:44:49--  http://us.download.nvidia.com/XFree86/Linux-x86_64/367.35/NVIDIA-Linux-x86_64-367.35.run
Resolving us.download.nvidia.com (us.download.nvidia.com)... 96.17.8.64, 96.17.8.50
Connecting to us.download.nvidia.com (us.download.nvidia.com)|96.17.8.64|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 75600336 (72M) [application/octet-stream]
Saving to: ‘NVIDIA-Linux-x86_64-367.35.run’

NVIDIA-Linux-x86_64 100%[===================>]  72.10M  69.9MB/s    in 1.0s    

2016-07-22 00:44:50 (69.9 MB/s) - ‘NVIDIA-Linux-x86_64-367.35.run’ saved [75600336/75600336]
{% endhighlight %}

Install some necessary packages before installing the NVIDIA GRID K520 driver
{% highlight css %}
$ sudo yum install gcc kernel-devel kernel-tools-devel xorg-x11-server-devel -y
{% endhighlight %}

Install the NVIDIA GRID K520 driver, be sure to specify the kernel source path. This is typically under `/usr/src/kernels/`
{% highlight css %}
$ chmod +x NVIDIA-Linux-x86_64-367.35.run
$ sudo ./NVIDIA-Linux-x86_64-367.35.run --kernel-source-path /usr/src/kernels/4.4.14-24.50.amzn1.x86_64/
Uncompressing NVIDIA Accelerated Graphics Driver for Linux-x86_64 367.35...........
{% endhighlight %}

After the Kernel modules are built, the NVIDIA GRID K520 driver installation process will start.

<center><iframe frameborder="0" height="480" src="https://www.scribd.com/embeds/385323752/content?start_page=1&view_mode=scroll&access_key=key-A2pMiGadX9wIGf8Lhspp&show_recommendations=true" width="640;"></iframe></center>

* Accept: License Agreement
* OK: For both of the X warnings
* Yes: Install the NVIDIA 32 bit compatibility libraries
* OK: For the the LibEGL.sc.1 warning
* No: run the nvidia-xconfig utility automatically

Enable the epel package manager so that p7zip can be installed.
{% highlight css %}
$ sudo yum-config-manager --enable epel
$ sudo yum install p7zip
{% endhighlight %}

Download, unzip and install Hashcat v3

{% highlight css %}
$ wget https://hashcat.net/files/hashcat-3.00.7z
--2016-07-22 00:56:52--  https://hashcat.net/files/hashcat-3.00.7z
Resolving hashcat.net (hashcat.net)... 149.154.152.149, 2a03:f80:ed15:149:154:152:149:1
Connecting to hashcat.net (hashcat.net)|149.154.152.149|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2687099 (2.6M) [application/x-7z-compressed]
Saving to: ‘hashcat-3.00.7z’

hashcat-3.00.7z             100%[==========================================>]   2.56M  1.22MB/s    in 2.1s    

2016-07-22 00:56:56 (1.22 MB/s) - ‘hashcat-3.00.7z’ saved [2687099/2687099]

$ 7za x hashcat-3.00.7z
$ rm hashcat-3.00.7z NVIDIA-Linux-x86_64-367.35.run
{% endhighlight %}

Start cracking hashes!
{% highlight css %}
$ cd hashcat-3.00/
$ ln -s hashcat64.bin hc
$ ./hc -b | tee hashcat–benchmark.txt
hashcat (v3.00-1-g67a8d97) starting in benchmark-mode...

OpenCL Platform #1: NVIDIA Corporation
======================================
- Device #1: GRID K520, 1009/4036 MB allocatable, 8MCU

Hashtype: MD4

Speed.Dev.#1.:  2428.9 MH/s (95.81ms)

Hashtype: MD5

Speed.Dev.#1.:  1724.2 MH/s (95.15ms)

Hashtype: Half MD5

Speed.Dev.#1.:  1288.9 MH/s (93.97ms)
...
{% endhighlight %}
