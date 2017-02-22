---
layout: post
title: Setting up BloodHound on Debian Jessie
date: '2016-10-06T11:57:00.000-06:00'
author: Sanjiv Kawa
tags:
- Debian
- Java
- graph
- neo4j
- BloodHound
- openjdk
comments: true
---
Every year I usually flag some tools that I want to try when I get back home from hacker summer camp. Things end up getting hectic, life and work takes over, and a year later, my reminders auto-delete rules remove all traces of what was once recorded.

Well not this year, I stuck to my guns and have been cycling through my list of "really cool tools to try".

I remember catching a bit of the BloodHound talk when I was presenting Wordsmith at BSides LV, and being quite excited to try it out.

To my surprise there isn't a good, concise guide to get BloodHound up and running on Debian. So here is what I was able to cobble together.

This should take about 15 minutes to install and configure.

First thing to do is sudo up.
{% highlight css %}
su root
{% endhighlight %}

Install some packages that will be necessary to grab BloodHound.
{% highlight css %}
apt-get update
apt-get install git wget curl
{% endhighlight %}

Add the neo4j sources.
{% highlight css %}
wget -O - https://debian.neo4j.org/neotechnology.gpg.key | sudo apt-key add -
echo 'deb http://debian.neo4j.org/repo stable/' | sudo tee /etc/apt/sources.list.d/neo4j.list
{% endhighlight %}

Add the openjdk 8 sources.
{% highlight css %}
echo "deb http://httpredir.debian.org/debian jessie-backports main" | sudo tee -a /etc/apt/sources.list.d/jessie-backports.list
{% endhighlight %}

Install openjdk 8.
{% highlight css %}
apt-get update
apt-get install openjdk-8-jdk openjdk-8-jre
{% endhighlight %}

This is **really** important, neo4j will run into a start up error unless the the default Java version is set to Java 8.
{% highlight css %}
root@seeker:~# java -version
java version "1.7.0_111"
OpenJDK Runtime Environment (IcedTea 2.6.7) (7u111-2.6.7-1~deb8u1)
OpenJDK Client VM (build 24.111-b01, mixed mode, sharing)
root@seeker:~# update-alternatives --config java
There are 2 choices for the alternative java (providing /usr/bin/java).

  Selection    Path                                           Priority   Status
------------------------------------------------------------
* 0            /usr/lib/jvm/java-7-openjdk-i386/jre/bin/java   1071      auto mode
  1            /usr/lib/jvm/java-7-openjdk-i386/jre/bin/java   1071      manual mode
  2            /usr/lib/jvm/java-8-openjdk-i386/jre/bin/java   1069      manual mode

Press enter to keep the current choice[*], or type selection number: 2
update-alternatives: using /usr/lib/jvm/java-8-openjdk-i386/jre/bin/java to provide /usr/bin/java (java) in manual mode
root@seeker:~# java -version
openjdk version "1.8.0_102"
OpenJDK Runtime Environment (build 1.8.0_102-8u102-b14.1-1~bpo8+1-b14)
OpenJDK Server VM (build 25.102-b14, mixed mode)
root@seeker:~#
{% endhighlight %}

Install neo4j.
{% highlight css %}
apt-get install neo4j
{% endhighlight %}

Edit the neo4j configuration file to listen for bolt and http on all interfaces (you can set 0.0.0.0 to 127.0.0.1 if you plan on using BloodHound for test only). Also also set the default graph database to graph.db.
{% highlight css %}
root@seeker:~# echo "dbms.active_database=graph.db" >> /etc/neo4j/neo4j.conf
root@seeker:~# echo "dbms.connector.http.address=0.0.0.0:7474" >> /etc/neo4j/neo4j.conf
root@seeker:~# echo "dbms.connector.bolt.address=0.0.0.0:7687" >> /etc/neo4j/neo4j.conf
root@seeker:~# tail /etc/neo4j/neo4j.conf
# Directory settings.
dbms.directories.logs=/var/log/neo4j
dbms.directories.run=/var/run/neo4j
dbms.directories.lib=/usr/share/neo4j/lib
dbms.directories.certificates=/var/lib/neo4j/certificates
dbms.active_database=graph.db
dbms.connector.http.address=0.0.0.0:7474
dbms.connector.bolt.address=0.0.0.0:7687
root@seeker:~#
{% endhighlight %}

The next step is to clone the Bloodhound git repository. This is just so we can extract the sample database. Once we place this into the appropriate location, we can delete the git repository.
{% highlight css %}
git clone https://github.com/adaptivethreat/BloodHound.git
cd BloodHound
cp -R /var/lib/neo4j/data/databases/graph.db /var/lib/neo4j/data/databases/graph.db.bak
cp -R BloodHoundExampleDB.graphdb/* /var/lib/neo4j/data/databases/graph.db
cd ..
rm -rf BloodHound
{% endhighlight %}

Restart neo4j.
{% highlight css %}
neo4j restart
{% endhighlight %}

Download the Bloodhound binary, but don't execute it just yet.
{% highlight css %}
wget https://github.com/adaptivethreat/BloodHound/releases/download/v1.1/BloodHound-linux-ia32.zip
unzip BloodHound-linux-ia32.zip
rm BloodHound-linux-ia32.zip
cd BloodHound-linux-ia32/
sudo chmod +x BloodHound
{% endhighlight %}

Before executing BloodHound, we have to set a new password for the neo4j instance.

Open a web browser and navigate to `http://yourDebianIp:7474`

<figure>
  <center>
    <img src="https://3.bp.blogspot.com/-eG3pGa7suBY/V_aOwBuTrZI/AAAAAAAACNs/VZi9pHR4iZEMHzCxEtDsV-sOgnJtkKjUQCLcB/s400/1.png">
    <figurecaption>Log in as: neo4j/neo4j</figurecaption>
  </center>
<figure>
<br><br>
<figure>
  <center>
    <img src="https://4.bp.blogspot.com/-kSxFkThI2Sw/V_aOwKTVv3I/AAAAAAAACNk/vS3shpnhc7s5zmK_VKLfn1_gw6J9sgEIQCEw/s1600/2.png">
    <figurecaption>Set new password.</figurecaption>
  </center>
<figure>
<br><br>
After setting a new password, feel free to close your browser and move back into your Debian instance to execute the BloodHound binary.

<figure>
  <center>
    <img src="https://1.bp.blogspot.com/-A8_LGW_Fvhc/V_aOweRnoLI/AAAAAAAACNw/J7fSo6ORs3YO3jVK2-ip_ItDjFjj-q6sQCEw/s400/4.png">s
    <figurecaption>This will launch the BloodHound native application. Set the database URL to: bolt://localhost:7687 and the database username and password to neo4j/yourNewPass</figurecaption>
  </center>
<figure>
<br><br>
<figure>
  <center>
    <img src="https://3.bp.blogspot.com/-SI0UXGzGazo/V_aOw1jLibI/AAAAAAAACN4/OX9VWLoN1kg0rv7s60RO6ooQ1H2pVN4lACEw/s400/5.png">
  </center>
<figure>
<br><br>
<figure>
  <center>
    <img src="https://1.bp.blogspot.com/-9SwCp3r5B7U/V_aOwpr9qPI/AAAAAAAACN0/LH_8DbBGHisiW6U_t4AtZ9h3wVkOZbQsgCEw/s400/6.png">
  </center>
<figure>
