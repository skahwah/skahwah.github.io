---
layout: post
title: Generating Revenue from Google AdSense
date: '2014-10-06T09:08:00.000-06:00'
author: Sanjiv Kawa
tags:
- Revenue Generation
- Monetization
- YouTube
comments: true
---
Recently I met with a friend who spoke about some people he knew generating large sums of money using a method which includes increasing views on YouTube videos. By no means am I advocating or endorsing this. I just think it would be interesting to look into as it has the potential to pose several security issues which we will get into later.

Before I get into anything, I want to mention that this is not a step by step guide to show you how to make money from YouTube videos. There are many resources on the internet that can show you how to do this both legally and illegally.

Lets get started.

Here is a real basic simplification about how AdSense and YouTube Monetization works:

1) You create a legitimate or fake Google account and link it with a Google AdSense account. Google AdSense is where all of your payment information goes into, and ultimately how you get paid.

2) By creating a Google account, you now have a YouTube account, you can then select to Monetize your account under Account Monetization. This effectively allows YouTube to place advertisements at the beginning of your video and on your video's page.

<figure>
  <center>
    <img src="https://3.bp.blogspot.com/-uUcvmlLSg3c/UkhJGxvLyaI/AAAAAAAABZA/aNCobThR-Os/s1600/Untitled.png">
    <figurecaption>I though this was pretty funny.</figurecaption>
  </center>
</figure>

3) After monetizing your YouTube account, you can then link it to your Google AdSense account and you will now be displaying revenue generating ad's on your YouTube content.
<figure>
  <center>
    <img src="https://2.bp.blogspot.com/-yGnmwVN0BZE/UkhJwndd2-I/AAAAAAAABZM/NC88mrQdvBo/s1600/Untitled2.png">
  </center>
</figure>

All of the initial set up for YouTube Monetization is really easy, the hard part is actually getting the views on your video.

There are various methods that people can use, some of them which are really easy such as sharing videos through social media, however to generate any real traffic you need to have a decent amount of legitimate followers on the social media accounts who are actually willing to click on your shared YouTube link.

Well, what if no one views the YouTube video? Are there other ways to increase traffic and view counts to my video? In short yes; illegally. I wanted to explore this area, and test it on an fake account that I created which is not linked up to Google Adsense but does the fundamentals of what we want, which is increase YouTube views on your uploaded video.

There is a bit of housekeeping we need to do first:

1) Create a test Google/YouTube account

2) Upload a video that is around 30 seconds to 1 minute long. This is completely arbitrary, it can be as long as you want really.

A method that I could think of which people would use to increase video views involved automation through programming and distributing copies of this program to bots/zombies. There are actual services which you can pay for that do this for you. So basically someone has created a program that would do the following:

Read in a a text document into the worker which would contain all of the revenue generating YouTube video links that the program should hit. Load it into an array so the infile is no longer needed as it would be computationally slower to read from.

Hit one of the video links for anywhere between 5 seconds to 35 seconds. From my understanding, the longer someone watches an advertisement, the more they get paid, YouTube doesn't really specify how much, but from what I was told it can be anywhere between $0.01 to $0.05 per video. This is why you create many, many arbitrary monetized videos and upload them to your account. These videos could be of your cat, or of gameplay or whatever. Additionally if you can write a program to actually click the ad, then you are winning.

Most ad's are around 30 seconds, so this way if you open the connection to your YouTube link for 35 seconds, you would stream the ad for 30 seconds and then start watching the video for 5 seconds before moving on to the next video in your list. Something that would work better would be to intelligently analyze how long the ad is, and then set your link timer based on that + how long you want to watch the actual video for.

This would then all be wrapped into a loop so that it would continuously cycle through all of the YouTube video links. I would also probably create some scheduled or random time breaks in the program so that it doesn't look obvious that this traffic is continuous and predictable.

We now have our program, but it's only coming from 1 computer. To really maximize generating revenue we want to spread this program across to different hosts, or spoof as many external IP's as possible. Any method could be used here, whether its a worm which drops the program as a payload throughout an university network or a bot controller pushing the program down to a bunch of zombie bots to run. It doesn't really matter how you do it.

Another function which I would write into the program would be to make it self destruct based on a command its sent, or on computer restart, or at a specified date. This is just for protection purposes. I would also heavily obfuscate the program.

I imagine that the hardest challenge would be to get a TCP stream open to YouTube which actually transfers video data and not just the data from the page. Sockets programming could help out here.

I cant think of may mitigation techniques which YouTube/Google could use to prevent automated views to videos, specifically accounts with monetization. There would have to be some sort of HTTP header challenge response that would need to be valid before proceeding with the TCP stream. Let me know what you guys think?
