---
layout: post
title: Staying clean with Vagrant and Chef
date: '2017-05-15T18:05:00.000-06:00'
author: Sanjiv Kawa
tags:
- Vagrant
- Chef
- Kali
- Penetration Testing
- VMware
- Packer
comments: true
---

## Intro
I've recently been thinking about the most effective way to sanitize my pen-testing VM's prior to the start of a new engagement. Reason being, pen-testing VM's can be quite unsanitary if neglected.

<img style="float: center;" src="https://raw.githubusercontent.com/skahwah/skahwah.github.io/master/_data/wash-hands.jpg" />

After the execution of most tools, confidential Client related data is stored in unknown/random locations, log files, or hidden directories. A good example of this are commands that are kept in history files that could contain credentials, asset information, or console output from Client systems.

My paranoia has only further been fueled by recent conference talks with the premise of blue team hunting the red team.

Up until now, I've been reverting to previous VM snapshots of "gold images" that are pre-configured with my preferred system configurations and tools. This is really effective, in the sense that it ensures that residual data from one pentest does not get introduced into the network for another pentest. However, it's not the most efficient in terms of manageability, and it also doesn't allow for dynamic changes such as the introduction of new tools or system and software updates.

As such, I've taken a page out of the DevOps playbook on virtual machine provisioning in my quest to create a safe, hygienic pen-testing environment.

### Main menu
In this blog post you will learn how to:
- Create a custom Vagrant base box for VMware
- Configure things like shared folders and bridged/NAT interfaces
- Modify system configurations and install packages via Chef
- Using Vagrant and Chef to create a sanitary pen-testing environment

### A word before we start
This blog post will largely be centered around paid tools, such as VMware Fusion and the Vagrant VMware plugin.

There are lots of existing blog posts that cover creating custom Vagrant boxes and provisioning them for VirtualBox, but surprisingly a handful of scattered sources for VMware.

Also, as I started playing more with VMware custom box creation, it became apparent that a lot of the native Vagrantfile configurations are geared for VirtualBox and not VMware. As such, I've made some creative hacks in the Vagrantfile to accommodate for the lack of native support. We'll get into this later, but for now, one example is:

Vagrant says to use the following configuration in the Vagrantfile to sync a folder from the virtual host to the virtual guest.
~~~ruby
config.vm.synced_folder "vagrant_data/", "/home/vagrant_data"
~~~

Well, this simply does not work when you are trying to create a custom box for VMware. Instead, I discovered that you can make direct modifications to the VMware vmx file and introduce folder sharing like this:
~~~ruby
config.vm.provider "vmware_fusion" do |v|
...
  v.vmx["sharedFolder0.present"] = "true"
  v.vmx["sharedFolder0.enabled"] = "true"
  v.vmx["sharedFolder0.readAccess"] = "true"
  v.vmx["sharedFolder0.writeAccess"] = "true"
  v.vmx["sharedFolder0.hostPath"] = "#{current_dir}/vagrant_data"
  v.vmx["sharedFolder0.guestName"] = "vagrant_data"
  v.vmx["sharedFolder0.expiration"] = "never"
...
~~~

### Requirements
Here are some of the tools that you'll need to follow along:

- Vagrant (Free)
- chefdk (Free)
- knife-solo (Free)
- Ruby >= 2.3.1 (Free)
- VMware Fusion (Paid)
- Vagrant VMware plugin (Paid)

## 1. Creating a custom VMware Vagrant box
As mentioned above, most of my battle was spent with creating a custom Vagrant Box for VMware. This is why I'm going to try and list as much detail as I possibly can so future VMware Vagranter's don't end up losing as much sleep as I did.

### 1.1 ISO Image
The custom VMware Vagrant box we'll be creating is a 64-bit version of Kali 2017.1. You can grab the ISO  [here](http://cdimage.kali.org/kali-2017.1/kali-linux-2017.1-amd64.iso).

There are a number of Kali boxes that exist in [HashiCorp's Atlas box catalog](https://atlas.hashicorp.com/boxes/search). A couple of reasons I elected to create my own box:
1. For fun - I wanted to learn how to create a Vagrant Box from scratch.
2. If this box is going to be in real Client environments, I want to know the contents of my box. In other words, I'm not trusting any pre-made boxes.

### 1.2 SSH key generation
Before creating a virtual instance of Kali we need to create an SSH key so Vagrant can interact with the virtual guest. You can use an existing key if you want, for the purposes of this blog post, I created a new SSH key pair on my host.

Keep note of the public key.
~~~
$ cd ~/.ssh
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/me/.ssh/id_rsa): vagrant-demo
$ cat vagrant-demo.pub
ssh-rsa AAAAB3N...
~~~

### 1.3 Creating the VMware virtual machine
Creating the VM is pretty standard. There are tools which automate this process, such as [Packer](https://www.packer.io/downloads.html). I found Packer works really well when creating a VirtualBox VM, but it struggles with certain things when creating a VM for VMware Fusion.

As such, I manually built my VM with the following specifications. Try to stick with these creation steps if you want a Vagrant box that has features like network interface selection and shared folder support.

Create a new VM from image with the assistant:
+ Select the Kali 2017.1 x64 ISO
+ Operating system: Debian 8.x 64-bit
+ Customize settings:
  + Sharing: Enable shared folders
  + Processors and memory: 2 CPU, 2048 MB of RAM
  + Network adapter: Share with my Mac (NAT)
  + Hard disk: 20 GB
  + USB & Bluetooth: Turn off "Share Bluetooth devices with Linux"
  + Remove sound card
  + Remove printer
  + Remove camera

Run through the installation process as normal. Once the VM boots into the desktop environment, navigate to the menu bar and click:

+ Virtual Machine > Install VMware Tools

This will mount VMware Tools into the virtual guests CD drive. Run the following:
~~~shell
$ apt-get update && apt-get upgrade -y
$ tar -xvf /media/cdrom0/VMwareT* -C /tmp
$ cd /tmp/VMware-tools-distrib/
$ ./VMware-install.pl
$ reboot
~~~

After the VM has finished rebooting, log in and run the following configuration script as root - replacing the public key with your own.
~~~shell
#!/bin/bash

publicKey="ssh-rsa AAAAB3N..."

mkdir ~/.ssh
echo "${publicKey}" > ~/.ssh/authorized_keys
chmod 700 ~/.ssh

mv /etc/ssh/sshd_config /etc/ssh/sshd_config.bak

cat >/etc/ssh/sshd_config <<EOL
Port 22
Protocol 2
HostKey /etc/ssh/ssh_host_rsa_key
HostKey /etc/ssh/ssh_host_dsa_key
HostKey /etc/ssh/ssh_host_ecdsa_key
HostKey /etc/ssh/ssh_host_ed25519_key
UsePrivilegeSeparation yes
KeyRegenerationInterval 3600
ServerKeyBits 1024
SyslogFacility AUTH
LogLevel INFO
LoginGraceTime 120
PermitRootLogin without-password
StrictModes yes
RSAAuthentication yes
PubkeyAuthentication yes
AuthorizedKeysFile	%h/.ssh/authorized_keys
IgnoreRhosts yes
RhostsRSAAuthentication no
HostbasedAuthentication no
PermitEmptyPasswords no
ChallengeResponseAuthentication no
PasswordAuthentication no
X11Forwarding yes
X11DisplayOffset 10
PrintMotd no
PrintLastLog yes
TCPKeepAlive yes
AcceptEnv LANG LC_*
UsePAM yes
EOL

update-rc.d ssh enable
service ssh start
~~~

Test SSH connectivity and shut down the virtual machine.
~~~shell
$ ssh -i ~/.ssh/vagrant-demo root@192.168.156.214

root@kali:~# poweroff
~~~

### 1.4 Convert a VMware VM to a Vagrant box
As opposed to VirtualBox, packaging a VMware VM into a Vagrant Box is more of a manual process. Again, [Packer](https://www.packer.io/downloads.html) can automate this, but works better for VirtualBox.

First, ensure that the recently created VM has been powered off. Check again.

Navigate to the location of the recently created Kali VM and create a file named `metadata.json`. This will tell Vagrant what provider this VM is for.
~~~shell
$ cd ~/Desktop/kali-2017-1-x64.vmwarevm/
$ echo '{"provider": "vmware_fusion"}'> metadata.json
$ cat metadata.json
{"provider": "vmware_fusion"}
$
~~~

Next, we are going to reduce the size of the Vagrant box. This will take about five minutes to complete.
~~~shell
$ cd ~/Desktop/kali-2017-1-x64.vmwarevm/
$ /Applications/VMware\ Fusion.app/Contents/Library/vmware-vdiskmanager -d Virtual\ Disk.vmdk
  Defragment: 100% done.
Defragmentation completed successfully.
$ /Applications/VMware\ Fusion.app/Contents/Library/vmware-vdiskmanager -k Virtual\ Disk.vmdk
  Shrink: 100% done.
Shrink completed successfully.
$
~~~

Lastly, we will compress the contents of the VM into an archive (box) that is readable by Vagrant. This can take about 10 minutes so feel free to grab a coffee.
~~~shell
$ tar -czvf kali-2017-x64.box ./*
...
$ mv kali-2017-x64.box ~/Desktop/
$ cd ~/Desktop/
~~~

## 2. Vagrant
At this point we no longer need the Kali VM that we created with VMware. Feel free to delete it.

The first thing we're going to do is check if the Vagrant VMware plugin has been added. Installation steps for adding this plugin to Vagrant can be found [here](https://www.vagrantup.com/docs/vmware/installation.html).
~~~shell
$ vagrant plugin list
vagrant-share (1.1.8)
  - Version Constraint: 1.1.8
vagrant-vmware-fusion (4.0.19)
~~~

Next, we'll add the recently created box into Vagrant. The value supplied to the name parameter can be set to anything you want.
~~~shell
$ cd ~/Desktop/
$ vagrant box add kali-2017-x64.box --name kali-2017-x64
$ vagrant box list
kali-2017-x64       (vmware_fusion, 0)
$
~~~

The box added into Vagrant exists as a base box, this is essentially the "gold image". Any modifications to a linked clone of this box will not be made to the base box itself.

Before bringing up a clone of the box, we need to create a couple of directories and make configurations to the Vagrantfile.
~~~
$ cd ~/Desktop/
$ mkdir -p kali-x64/vagrant_data
$ cd kali-x64/
$ vagrant init
$ ls
Vagrantfile  vagrant_data
$
~~~

A Vagrantfile is created once the `vagrant init` command is executed. Think of this file as a blank canvas that we can use to configure our cloned Vagrant box.

Below is a copy of the Vagrantfile that I am using for the kali-2017-x64 box. The `config.ssh.private_key_path` will need to be changed to your SSH private key.

Something else to note is the networking capabilities of this Vagrantfile. Currently, the network interface is configured in NAT mode. If the comment tag is removed from the line that reads `v.vmx["ethernet0.connectiontype"] = "bridged"`, then the Vagrant box will be on the same network as your host system and assigned an IP address by DHCP (assuming that DHCP is enabled on the local network). I have not experimented with [Vagrant port-forwarding configurations](https://www.vagrantup.com/docs/networking/forwarded_ports.html) in VMware. I anticipate that these will need to be made in the vmx configuration given the previous issues I have experienced.
~~~ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

current_dir = File.dirname(__FILE__)

Vagrant.configure("2") do |config|
  config.vm.box = "kali-2017-x64"
  config.ssh.username = "root"
  config.ssh.private_key_path = "/Users/me/.ssh/vagrant-demo"

  config.vm.provider "vmware_fusion" do |v|
    v.gui = true
    v.vmx["memsize"] = "2048"
    v.vmx["numvcpus"] = "2"
    v.vmx["displayName"] = "Kali 2017.1 x64"
    v.vmx["ide1:0.filename"] = ""
    v.vmx["sharedfolder.maxnum"] = "1"

    # shared folders
    v.vmx["sharedFolder0.present"] = "true"
    v.vmx["sharedFolder0.enabled"] = "true"
    v.vmx["sharedFolder0.readAccess"] = "true"
    v.vmx["sharedFolder0.writeAccess"] = "true"
    v.vmx["sharedFolder0.hostPath"] = "#{current_dir}/vagrant_data"
    v.vmx["sharedFolder0.guestName"] = "vagrant_data"
    v.vmx["sharedFolder0.expiration"] = "never"

    # networking
    # NAT is enabled if bridged is commented out
    # v.vmx["ethernet0.connectiontype"] = "bridged"
  end
end
~~~

Once the configurations have been made to the Vagrantfile we can `vagrant up`. This spins up a linked clone of the kali-2017-x64 Vagrant box. There will be several warnings which are safe to ignore. Including the rather verbose message at the end which reads "*Vagrant attempted to execute the capability ...*".
~~~shell
$ cd ~/Desktop/kali-x64/
$ vagrant up
Bringing machine 'default' up with 'vmware_fusion' provider...
==> default: Cloning VMware VM: 'kali-2017-x64'. This can take some time...
==> default: Verifying vmnet devices are healthy...
==> default: Preparing network adapters...
WARNING: The VMX file for this box contains a setting that is automatically overwritten by Vagrant
WARNING: when started. Vagrant will stop overwriting this setting in an upcoming release which may
WARNING: prevent proper networking setup. Below is the detected VMX setting:
WARNING:
WARNING:   ethernet0.pcislotnumber = "33"
WARNING:
WARNING: If networking fails to properly configure, it may require this VMX setting. It can be manually
WARNING: applied via the Vagrantfile:
WARNING:
WARNING:   Vagrant.configure(2) do |config|
WARNING:     config.vm.provider :vmare_fusion do |vmware|
WARNING:       vmware.vmx["ethernet0.pcislotnumber"] = "33"
WARNING:     end
WARNING:   end
WARNING:
WARNING: For more information: https://www.vagrantup.com/docs/vmware/boxes.html#vmx-whitelisting
==> default: Starting the VMware VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 192.168.156.215:22
    default: SSH username: root
    default: SSH auth method: private key
==> default: Machine booted and ready!
==> default: Forwarding ports...
    default: -- 22 => 2222
==> default: Configuring network adapters within the VM...
Vagrant attempted to execute the capability 'configure_networks'
on the detect guest OS 'linux', but the guest doesnt
support that capability. This capability is required for your
configuration of Vagrant. Please either reconfigure Vagrant to
avoid this capability or fix the issue by creating the capability.
$
~~~

After spinning up the kali-2017-x64 Vagrant box, we can `vagrant ssh` into it and start interacting with the box. In the example below, we create a testfile in a shared folder from our host that is observable in our Vagrant box.
~~~shell
$ vagrant ssh

root@kali:~# cd /mnt/hgfs/vagrant_data/
root@kali:/mnt/hgfs/vagrant_data# touch testfile
root@kali:/mnt/hgfs/vagrant_data# exit
logout
Connection to 192.168.156.215 closed.
$ cd vagrant_data/
$ ls
testfile
$
~~~

It is also **important** to take note of the release description for later use. This should be "Kali GNU/Linux Rolling", but could be different if you're reading this blog and using another custom operating system build.
~~~
$ vagrant ssh

root@kali:~# cat /etc/*release | grep DESC
DISTRIB_DESCRIPTION="Kali GNU/Linux Rolling"
root@kali:~# exit
logout
Connection to 192.168.156.215 closed.
~~~

Once the box has been exited the virtual instance is still running.
~~~shell
$ vagrant global-status  
id       name    provider      state   directory
---------------------------------------------------------------------------
3c18ff0  default vmware_fusion running /Users/me/Desktop/kali-x64
...
~~~

We can use `vagrant halt` to shut down the VM or `vagrant suspend` to suspend the VM. We'll leave it up and running for now.

At this point, we have successfully created a custom Vagrant box from a VMware VM!

## 3. Chef
We need to do a little housekeeping before we can start provisioning our Vagrant box with Chef.

### 3.1 Housekeeping
The first thing we'll need to do is install [chefdk](https://downloads.chef.io/chefdk#mac_os_x).

After that, ensure that Ruby >=2.3.1 is installed on your host. You can use the following instructions if you have the homebrew package manager installed.
~~~shell
$ brew install rbenv ruby-build
$ echo 'if which rbenv > /dev/null; then eval "$(rbenv init -)"; fi' >> ~/.bash_profile
$ source ~/.bash_profile
$ rbenv install 2.3.1
$ rbenv global 2.3.1
$ ruby -v
~~~

We're then going to install a couple of Ruby gems, most notably knife-solo which will allow us to install Chef and use Chef to interact with our Vagrant box.
~~~shell
$ gem install net-ssh -v 3.2.0
$ gem install net-ssh knife-solo
~~~

After knife-solo has been installed, we need to make an edit to `/Users/me/.rbenv/versions/2.3.1/lib/ruby/gems/2.3.0/gems/knife-solo-0.5.1/lib/knife-solo/bootstraps/linux.rb`.

We are concerned with the `distro` function located on line 70 and want to add the `Kali GNU/Linux Rolling` case statement:
~~~ruby
def distro
  return @distro if @distro
  @distro = case issue
  when %r{Kali GNU/Linux Rolling}
    {:type => "debianoid_gem"}
  when %r{Debian GNU/Linux [678]}
    {:type => (x86? ? "debianoid_omnibus" : "debianoid_gem")}
  when %r{Debian}
    {:type => "debianoid_gem"}
  when %r{Raspbian}
    {:type => "debianoid_gem"}
  when %r{Linux Mint}
    ...
~~~

This will tell knife-solo to treat a release that has the description "Kali GNU/Linux Rolling" as Debian.

That should wrap up all of the housekeeping steps.

### 3.2 Creating Chef cookbooks
I'm going to preface this section by saying that I am relatively new to Chef and could be doing this completely wrong. Please correct me if you think I can do this better. From my understanding, the organization of Chef kitchens is similar to the following:
~~~shell
chef-kitchen
-> cookbooks
  -> cookbook_for_updates
    -> recipe
  -> cookbook_for_git_repos
    -> recipe
  -> cookbook_for_system_configs
    -> recipe
  -> cookbook_for_xyz
    -> recipe
-> nodes
  -> 192.168.1.1
    -> recipe[cookbook_for_updates, cookbook_for_system_configs]
  -> 192.168.1.2
    -> recipe[cookbook_for_updates, cookbook_for_git_repos, cookbook_for_system_configs, cookbook_for_xyz]
  -> 192.168.1.13
    -> recipe[cookbook_for_system_configs, cookbook_for_xyz]
~~~

We'll navigate back to our Vagrant kali-x64 directory and create a Chef kitchen named kali_kitchen.
~~~shell
$ cd ~/Desktop/kali-x64/
$ knife solo init kali_kitchen
~~~

We'll then create a cookbook within this kitchen called system_configurations/
~~~shell
$ cd kali_kitchen/cookbooks
$ chef generate cookbook system_configurations
~~~

Within the kali_kitchen, we can specify nodes, which in our case are Vagrant boxes, that will receive cookbooks created in this kitchen. Additionally, these nodes can receive some cookbooks or all of the cookbooks. The choice is yours.

Next, we'll create a recipe for the the system_configurations cookbook. This takes place by editing the `default.rb` file for the system_configurations cookbook. In the example below, we create a directory named "tools" and create a "bash_aliases" file.

~~~ruby
$ cat system_configurations/recipes/default.rb

#
# Cookbook:: system_configurations
# Recipe:: default
#
# Copyright:: 2017, Sanjiv Kawa, All Rights Reserved.

directory '/tools/' do
  owner 'root'
  group 'root'
  mode '0755'
  action :create
end

ret = "\n"
aliases = "alias c='clear'" + ret +
"alias cdd='cd /Users/root/Desktop'" + ret +
"alias grip=\"grep -Eo \'[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}\'\"" + ret +
"alias l='ls -hal'" + ret +
"alias sau='sort | uniq'" + ret +

file "/root/.bash_aliases" do
  content aliases
end

script "system_configurations" do
  interpreter "bash"
  code <<-EOH
    echo 'set completion-ignore-case On' >> ~/.inputrc
    source /root/.bashrc
    EOH
end

~~~

### 3.3 Provisioning Vagrant boxes with Chef
Once changes have been made to our system_configurations cookbook, we'll navigate back to our Chef kitchen in the Vagrant kali-x64 directory and prepare the Vagrant box. This will do several things such as, install the Chef client on the Vagrant box and make it a member node of our kali_kitchen.
~~~shell
$ cd ~/Desktop/kali-x64/kali_kitchen
$ knife solo prepare --identity-file ~/.ssh/vagrant-demo root@192.168.156.215
Bootstrapping Chef...
Updating apt caches...
Installing required packages...
Installing rubygems from source...
Generating node config 'nodes/192.168.156.215.json'...
~~~

As seen above, our Vagrant box is now part of kali_kitchen. We can now apply the system_configurations cookbook to the Vagrant box by editing the node's `run_list`. This node will now receive the system_configurations recipe, previously defined in the `default.rb` file.
~~~shell
$ cat nodes/192.168.156.215.json
{
  "run_list": [
    "recipe[system_configurations]"
  ],
  "automatic": {
    "ipaddress": "192.168.156.215"
  }
}
~~~

After editing the `run_list` node configuration, we can "cook" the recipe on our Vagrant box.
~~~shell
$ cd ~/Desktop/kali-x64/kali_kitchen
$ knife solo cook --identity-file ~/.ssh/vagrant-demo root@192.168.156.215
Running Chef on 192.168.156.215...
Uploading the kitchen...
Generating solo config...
Running Chef: sudo chef-solo -c ~/chef-solo/solo.rb -j ~/chef-solo/dna.json
Starting Chef Client, version 13.0.118
resolving cookbooks for run list: ["system_configurations"]
Synchronizing Cookbooks:
  - system_configurations (0.1.0)
Installing Cookbook Gems:
Compiling Cookbooks...
Converging 3 resources
Recipe: system_configurations::default
  * directory[/tools/] action create
  - create new directory /tools/
  - change mode from '' to '0755'
  - change owner from '' to 'root'
  - change group from '' to 'root'
  * file[/root/.bash_aliases] action create (up to date)
  * script[system_configurations] action run
    - execute "bash"  "/tmp/chef-script20170515-10727-op1gwo"

Running handlers:
Running handlers complete
Chef Client finished, 1/3 resources updated in 02 seconds
$
~~~

Our Vagrant box has now had various system configurations applied to it via Chef.
~~~shell
root@kali:~# alias
alias c='clear'
alias cdd='cd /Users/root/Desktop'
alias grip='grep -Eo '\''[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}'\'''
alias l='ls -hal'
alias ls='ls --color=auto'
alias sau='sort | uniq'
alias search='reset; grep -H'
root@kali:~# ls / | grep -i too
tools
root@kali:~#
~~~

### 3.4 Other cookbook thoughts
I have some cookbook ideas and over time will be adding them to [my GitHub page](https://github.com/skahwah/chef). This will be things like:
- A cookbook containing all of the tools I frequently use, this will download the tools into the /tools directory then install them
- A cookbook for updates and upgrades that runs periodically

## 4. Staying clean with Vagrant and Chef
All of this seems like a lot of work, but once your base box is created and your Chef cookbooks have been made, it's really easy to rapidly provision systems prior to the start of an engagement. In addition, small Vagrant files result in your penetration testing environment becoming a lot smaller and the introduction of cookbooks make it completely portable.

Let's create a scenario. It's Friday, you've collected all of your evidence and the pen-testing gig you were on to is now over. You can rip down your entire Vagrant environment with a single command:
~~~shell
root@kali:~# exit
logout
Connection to 192.168.156.215 closed.
$ vagrant destroy
    default: Are you sure you want to destroy the 'default' VM? [y/N] y
==> default: Stopping the VMware VM...
==> default: Deleting the VM...
$
~~~

You can also delete any files you have copied into to your shared directory to ensure that there will be no cross-contamination or stray Client files:
~~~shell
$ rm -rf vagrant_data && mkdir vagrant_data
~~~

Monday comes around and you need a fresh environment for new pen-testing gig. This can be achieved in a just a couple of commands:
~~~shell
$ vagrant up
$ knife solo prepare --identity-file ~/.ssh/vagrant-demo root@192.168.156.214
$ cd kali_kitchen
$ vim nodes/192.168.156.214.json
$ knife solo cook --identity-file ~/.ssh/vagrant-demo root@192.168.156.214
$ vagrant ssh
~~~
