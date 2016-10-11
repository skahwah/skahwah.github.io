---
layout: post
title: Using Postfix and OSSEC to Monitor Your VPS or Production *NIX Systems
date: '2015-11-06T17:52:00.000-07:00'
author: Sanjiv Kawa
tags:
- Monitoring
- Linux
- OSSEC
- Security
- Alerts
- Postfix
comments: true
---
A few months ago I was experimenting with receiving custom alerts from my internet facing \*NIX boxes. Some of the things I wanted to monitor in real-time (or close enough to) were successful SSH login attempts, sudo/privilege escalation alerts and new file creations. I wrote a few shell scripts that handled this for me and it ended up working pretty well.

Last week I started looking tools that handle monitoring and alerting and stumbled upon <a href="http://www.ossec.net/">OSSEC</a>. OSSEC encompasses everything I was looking to receive alerts for and more. For those of you who aren't familiar with OSSEC, it's an *"Open Source Host-based Intrusion Detection System that performs log analysis, file integrity checking, policy monitoring, rootkit detection, real-time alerting and active response."*

After playing around with OSSEC for a few days now I've come to a pretty solid conclusion that it's going to be a regular tool that will be featuring in my \*NIX builds from now on.

I decided to put together this guide to help out people who want a little more insight into what is happening on their VPS or production \*NIX systems.

# 0. Pre-requisites
For this guide in particular you're going to need:

* A VPS running Ubuntu
* DNS configured to point to the public IP address of the VPS.
* A mail account with Google or a similar provider.

Throughout this guide I'll refer to the VPS as `sub.domain.com` and the mail account as `me@gmail.com`.

# 1. Installing and Configuring Postfix as a Send-Only SMTP Server
Postfix is going to handle all of the SMTP relaying for us, you don't need to open any TCP or UDP ingress ports.

First, make sure that the hostname of your VPS is configured correctly.
{% highlight css %}
sudo hostname sub.domain.com
{% endhighlight %}

Next, update the box and install mailutils

{% highlight css %}
sudo apt-get update && sudo apt-get install mailutils -y
{% endhighlight %}

As the mailtutils installation progresses, you will reach a point where you will need to specify the Postfix configuration type. Select `Internet Site`.

<figure>
  <center>
    <img src="https://3.bp.blogspot.com/-Onpr7pUuVPM/VjwTBD5EgaI/AAAAAAAAB8o/Iz-Y5Xj513Q/s400/Screen%2BShot%2B2015-11-05%2Bat%2B7.38.59%2BPM.png">
  </center>
</figure>

You should then be prompted by the Postfix configuration agent to enter the mail name for the system.

<figure>
  <center>
    <img src="https://3.bp.blogspot.com/-zOQ5h7X44iE/VjwYJ0KL5HI/AAAAAAAAB88/CbxfrrRNm-0/s1600/Screen%2BShot%2B2015-11-05%2Bat%2B8.01.32%2BPM.png">
  </center>
</figure>

Next, you'll need to edit the main configuration file for postfix to ensure that the internet interface is set as the localhost. This should be the second last line of the file:
{% highlight css %}
sudo cp /etc/postfix/main.cf /etc/postfix/main.cf.bak
sudo vim /etc/postfix/main.cf

...

# inet_interfaces = all
inet_interfaces = localhost
inet_protocols = all
{% endhighlight %}

That should be all of the configurations needed to configure Postfix as a send-only SMTP server. Restart Postfix and send a test email to your Google mail account by running the following mail commands:

{% highlight css %}
sudo service postfix restart
echo "This is the body of the email" | mail -s "This is the subject line" me@gmail.com
{% endhighlight %}

You should receive an email similar to the following:
<figure>
  <center>
    <img src="https://2.bp.blogspot.com/-0MUDj-zjG4M/VjwZvhRoj7I/AAAAAAAAB9I/dhxZZlL6CQw/s400/1.png">  
  </center>
</figure>

Next, let's set up some mail aliases so that whenever users such as "root" or "ubuntu" receive mail, it is forwarded to your Google mail account.

{% highlight css %}
sudo vim /etc/aliases
{% endhighlight %}

Append the following lines to /etc/aliases:
{% highlight css %}
root:          me@gmail.com
ubuntu:        me@gmail.com
{% endhighlight %}

Commit the changes to /etc/aliases and send a test email to "root" and "ubuntu" by running the following mail commands:
{% highlight css %}
sudo newaliases
echo "This is the body of the email from root" | mail -s "This is the subject line from root" root
echo "This is the body of the email from ubuntu" | mail -s "This is the subject line from ubuntu" ubuntu
{% endhighlight %}

You should receive an email similar to the following:
<figure>
  <center>
    <img src="https://2.bp.blogspot.com/-weo-LwvM5z8/VjwcLjZXQ3I/AAAAAAAAB9U/55jxjQAgXiY/s1600/2.png">
  </center>
</figure>

# 2. Installing OSSEC
First, you'll need to update and install some essential packages and notification tools that are required for real-time alerting to work with OSSEC.
{% highlight css %}
sudo apt-get update && sudo apt-get install build-essential inotify-tools
{% endhighlight %}

Next, download and install OSSEC.
{% highlight css %}
cd && wget -U ossec https://bintray.com/artifact/download/ossec/ossec-hids/ossec-hids-2.8.3.tar.gz
tar -zxf ossec-hids-2.8.3.tar.gz && cd ossec-hids-2.8.3
sudo ./install.sh
{% endhighlight %}

As you are running through the installation, ensure that you enter the following when prompted:

{% highlight css %}
1- What kind of installation do you want (server, agent, local, hybrid or help)? local

  - Local installation chosen.

2- Setting up the installation environment.

 - Choose where to install the OSSEC HIDS [/var/ossec]:

    - Installation will be made at  /var/ossec .

3- Configuring the OSSEC HIDS.

  3.1- Do you want e-mail notification? (y/n) [y]: y
   - What's your e-mail address? root@localhost      

   - We found your SMTP server as: 127.0.0.1
   - Do you want to use it? (y/n) [y]: y

   --- Using SMTP server:  127.0.0.1

  3.2- Do you want to run the integrity check daemon? (y/n) [y]: y

   - Running syscheck (integrity check daemon).

  3.3- Do you want to run the rootkit detection engine? (y/n) [y]: y

   - Running rootcheck (rootkit detection).

  3.4- Active response allows you to execute a specific
       command based on the events received. For example,
       you can block an IP address or disable access for
       a specific user.  
       More information at:
       http://www.ossec.net/en/manual.html#active-response

   - Do you want to enable active response? (y/n) [y]: y

     - Active response enabled.

   - By default, we can enable the host-deny and the
     firewall-drop responses. The first one will add
     a host to the /etc/hosts.deny and the second one
     will block the host on iptables (if linux) or on
     ipfilter (if Solaris, FreeBSD or NetBSD).
   - They can be used to stop SSHD brute force scans,
     portscans and some other forms of attacks. You can
     also add them to block on snort events, for example.

   - Do you want to enable the firewall-drop response? (y/n) [y]: y

     - firewall-drop enabled (local) for levels >= 6

   - Default white list for the active response:
      - x.x.x.x

   - Do you want to add more IPs to the white list? (y/n)? [n]: n

  3.5- Do you want to enable remote syslog (port 514 udp)? (y/n) [y]: y

   - Remote syslog enabled.

  3.6- Setting the configuration to analyze the following logs:
    -- /var/log/auth.log
    -- /var/log/syslog
    -- /var/log/dpkg.log

 - If you want to monitor any other file, just change
   the ossec.conf and add a new localfile entry.
   Any questions about the configuration can be answered
   by visiting us online at http://www.ossec.net.
{% endhighlight %}

Check the status of OSSEC and ensure that it is **not** running:

sudo /var/ossec/bin/ossec-control status
{% highlight css %}
ossec-monitord not running...
ossec-logcollector not running...
ossec-syscheckd not running...
ossec-analysisd not running...
ossec-maild not running...
ossec-execd not running...
{% endhighlight %}

# 3. Configuring OSSEC
Most if not all of OSSEC’s files are contained within `/var/ossec`.

OSSEC binaries are in `/var/ossec/bin`
Configuration files are in `/var/ossec/etc`
Rules are contained within `/var/ossec/rules`

The OSSEC configuration file is located at `/var/ossec/etc/ossec.conf`. This holds all settings broken down into several XML elements.

The first thing you'll need to do is create a backup copy of the OSSEC main configuration file.

{% highlight css %}
sudo -i
cd /var/ossec/etc/
cp ossec.conf ossec.conf.bak
vim ossec.conf
{% endhighlight %}

After opening and scrolling through the configuration file, you will notice that there is a `<frequency>` element nested under `<syscheck>`. This should contain a value of `79200`. This value refers to how many times `<syscheck>` is executed within a certain amount of seconds. `<syscheck>` is responsible for scanning the file system to identify new files. For testing purposes you can set the value of `<frequency>` to `60`, which is 60 seconds. After you feel that testing has concluded you can set this value back to `79200` which is 22 hours or `28800` which is 8 hours.

`<frequency>60</frequency>
`
Personally, I like to be notified when new files are created, to configure this option, navigate down to the `<syscheck> `section and add the following lines under the `<frequency>` element.

{% highlight css %}
<!-- Added: alert_new_files to send a notification each time a new file is created -->
<alert_new_files>yes</alert_new_files>
{% endhighlight %}

By default, OSSEC monitors the following directories and nested files which are defined in the `<directories>` element.

`/etc`<br>
`/usr/bin`<br>
`/usr/sbin`<br>
`/bin`<br>
`/sbin`<br>

You can enable realtime monitoring on these directories by adding the `report_changes` and `realtime` attributes and corresponding `yes` values within the `<directories>` element.
{% highlight css %}
<directories report_changes="yes" realtime="yes" check_all="yes">/etc,/usr/bin,/usr/sbin</directories>
<directories report_changes="yes" realtime="yes" check_all="yes">/bin,/sbin</directories>
{% endhighlight %}

I would recommend identifying key areas of your file system that you want monitored before adding the directory in the `<directories>` element. For example, you wouldn’t want to monitor the `/var/logs `directory as files within this directory are continuously changing and you will receive emails from the server. OSSEC sends a maximum of 12 emails per hour (you can increase or decrease this). Some additional directories you may want to add are:

`/tmp`<br>
`/home/ubuntu`<br>
`/home/your-user-name`<br>
`/root`<br>

If you have a CMS such as WordPress installed, these directories may be something you would want to include within OSSEC’s monitoring scope.

`/var/www/html/wp-content/uploads`<br>
`/var/www/html/wp-content/plugins`<br>

The rules directory is located at `/var/ossec/rules/` and houses all of the rules which helps OSSEC determine its course of action for various behaviors that occur on the file system and exposed services.

The last thing you'll need to do is configure a local rule so that OSSEC sends an alert if a file change has been detected. Create a backup copy of the OSSEC `local_rules.xml` file

{% highlight css %}
sudo cp /var/ossec/rules/local_rules.xml /var/ossec/rules/local_rules.xml.bak
{% endhighlight %}

As we want to receive alerts for new files that are added to the system we can edit `/var/ossec/rules/local_rules.xml` and append the following before the `</group>` tag:
{% highlight css %}
<rule id="554" level="7" overwrite="yes">
    <category>ossec</category>
    <decoded_as>syscheck_new_entry</decoded_as>
    <description>File added to the system.</description>
    <group>syscheck,</group>
</rule>
{% endhighlight %}

This should look similar to the following:
<figure>
  <center>
    <img src="https://3.bp.blogspot.com/-nDlbLPJ2XdM/Vj0-K3Hs0FI/AAAAAAAAB98/LDDLBvhznoQ/s1600/Screen%2BShot%2B2015-11-06%2Bat%2B4.53.40%2BPM.png">
  </center>
</figure>

After you have saved the `/var/ossec/rules/local_rules.xml` file, we can go ahead and start OSSEC.

{% highlight css %}
/var/ossec/bin/ossec-control start
{% endhighlight %}

Starting OSSEC should look similar to the following:
<figure>
  <center>
    <img src="https://4.bp.blogspot.com/-npDBEFTqx_A/Vj0_EY02axI/AAAAAAAAB-E/oIZdhqQhcZQ/s1600/Screen%2BShot%2B2015-11-06%2Bat%2B4.57.30%2BPM.png">
  </center>
</figure>

You can check the status of OSSEC by executing the following command.

{% highlight css %}
/var/ossec/bin/ossec-control status
{% endhighlight %}

If you experience any issues trying to get OSSEC to start, check out the log file located at `/var/ossec/logs/ossec.log`

After starting OSSEC, you should receive an email similar to this:

<figure>
  <center>
    <img src="https://2.bp.blogspot.com/-UdcS0-3r11E/Vj1AAQRc1qI/AAAAAAAAB-M/6ESTSOSmFlg/s400/2.png">
  </center>
</figure>

Next, let's create some test scenarios to make sure that alerting through OSSEC and Postfix is working correctly. Create a file named `ossec-test.conf` in `/etc` and write a string of text into it:

{% highlight css %}
cd /etc
vim ossec-test.conf
write some text into ossec-test.conf
{% endhighlight %}

You should receive an alert similar to this:
<figure>
  <center>
    <img src="https://2.bp.blogspot.com/-pqxvPw9sC1A/Vj1Iam-wr_I/AAAAAAAAB-w/NLl8BX8UDjc/s400/4.png">
  </center>
</figure>

Exit your SSH session and log back into the server. You should receive an email similar to the following:
<figure>
  <center>
    <img src="https://4.bp.blogspot.com/-xfiJDYHXEQM/Vj1DcfIyRkI/AAAAAAAAB-Y/tAAGmbZvzbo/s400/3.png">
  </center>
</figure>

To wrap things up, make sure that you go back to your OSSEC configuration file and set the `<frequency>` value back to `79200` which is 22 hours or `28800` which is 8 hours. Restart OSSEC for the new configurations to be loaded:

{% highlight css %}
/var/ossec/bin/ossec-control restart
{% endhighlight %}

Delete the test files that were created:

{% highlight css %}
rm /etc/ossec-test.conf
{% endhighlight %}
