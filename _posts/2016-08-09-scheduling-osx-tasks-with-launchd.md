---
layout: post
title: Scheduling OSX tasks with launchd
date: '2016-08-09T18:14:00.000-06:00'
author: Sanjiv Kawa
tags:
- OSX
- plist
- home brew
- scheduled task
- Apple
- launch agent
- launchctl
- brew
- launchd
comments: true
---
Scheduling system tasks on OSX can be achieved with either with cron or launchd. At a first glance, it looks like launchd supersedes cron on OSX and when a job is created in cron, launchd actually does all the work.

Anyways, I decided to have a crack at creating a scheduled task through launchd which updates brew for me.

Creating a scheduled task is fairly straightforward. This consists of creating a launch agent, which is a structured plist file that looks a lot like XML. All user launch agents are stored in `~/Library/LaunchAgents` and run in context of the currently logged in user. These agents are all automatically loaded on boot. For more info on launchd, check out this <a href="https://www.blogger.com/blogger.g?blogID=6624093094976579373">guide</a>.

The plist I created is named `local.brewupdate.job.plist`, it basically runs brew with the update argument everyday at 2pm. After the job runs, two files are created in `/tmp`. `/tmp/local.brewupdate.job.stdout` writes out what is displayed to console when brew update is run. This is typically a list of updated programs, taps, etc. If a launch error occurs, specific debug messages are written to `/tmp/local.brewupdate.job.stderr`.

{% highlight css %}
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key>
  <string>local.brewupdate.job</string>
  <key>ProgramArguments</key>
  <array>
    <string>/usr/local/bin/brew</string>
    <string>update</string>
  </array>
  <key>StandardErrorPath</key>
  <string>/tmp/local.brewupdate.job.stderr</string>
  <key>StandardOutPath</key>
  <string>/tmp/local.brewupdate.job.stdout</string>
  <key>StartCalendarInterval</key>
  <array>
    <dict>
      <key>Hour</key>
      <integer>14</integer>
      <key>Minute</key>
      <integer>0</integer>
    </dict>
  </array>
</dict>
</plist>
{% endhighlight %}

Create the launch agent
{% highlight css %}
17:19 skawa@skawa-mbp: ~ $ cd Library/LaunchAgents
17:19 skawa@skawa-mbp: LaunchAgents $ pwd
/Users/skawa/Library/LaunchAgents
17:19 skawa@skawa-mbp: LaunchAgents $ vim local.brewupdate.job.plist
{% endhighlight %}

Load the launch agent
{% highlight css %}
17:19 skawa@skawa-mbp: LaunchAgents $ launchctl list | grep brew
17:19 skawa@skawa-mbp: LaunchAgents $ launchctl load local.brewupdate.job.plist
17:19 skawa@skawa-mbp: LaunchAgents $ launchctl list | grep brew
- 0 local.brewupdate.job
{% endhighlight %}

Test if the job runs correctly
{% highlight css %}
17:19 skawa@skawa-mbp: LaunchAgents $ cat /tmp/local.brewupdate.job.std*
cat: /tmp/local.brewupdate.job.std*: No such file or directory
17:19 skawa@skawa-mbp: LaunchAgents $ launchctl start local.brewupdate.job
17:19 skawa@skawa-mbp: LaunchAgents $ ls /tmp/local.brewupdate.job.std*
local.brewupdate.job.stderr  local.brewupdate.job.stdout  
17:19 skawa@skawa-mbp: LaunchAgents $ cat /tmp/local.brewupdate.job.stdout
Already up-to-date.
17:19 skawa@skawa-mbp: LaunchAgents $ launchctl stop local.brewupdate.job
17:19 skawa@skawa-mbp: LaunchAgents $ launchctl list | grep brew
- 0 local.brewupdate.job
17:19 skawa@skawa-mbp: LaunchAgents $ rm /tmp/local.brewupdate.job.std*
17:20 skawa@skawa-mbp: LaunchAgents $
{% endhighlight %}
