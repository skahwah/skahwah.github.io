---
layout: post
title: PWK Course and OSCP Exam Review
date: '2015-04-08T12:31:00.000-06:00'
author: Sanjiv Kawa
tags:
- Offensive Security
- PWK
- Lab
- OSCP
- Kali
- Penetration Testing
- Course
- Review
comments: true
---
# 0x00 - Starting Off
The Offensive Security Certified Professional (OSCP) certification is by far the most challenging and the most rewarding achievement I have accomplished. For those of you who aren't familiar with the OSCP, it is the worlds first completely hands on information security certificate. This means that there is no theory in this course, no study guide and no multiple choice exam. Instead, students are given access to the Penetration Testing with Kali (PWK) labs to develop their pen-testing skills before attempting to challenge the excruciatingly painful 24 hour OSCP exam. This is then followed by an additional 24 hours to compose and submit a formal penetration test report of the OSCP exam lab. But we'll touch more on that later.

# 0x01 - My Experience Before Starting
I have about 3 years of technical experience in information security which certainly contributed to my success in obtaining the OSCP. That being said, I would of considered myself a junior prior to taking this course. Some of my involvement in information security has consisted of conducting vulnerability assessments and penetration tests for medium and large sized organizations. One of the things that I came realize over the duration of this course is how important the methodological approach to penetration testing actually is. If I could go 6 months back in time I would tell myself to enumerate more and to not stop enumerating until everything that is even remotely enumerable has been enumerated.

<figure>
  <center>
    <img src="https://i.imgflip.com/jwevp.jpg">
  </center>
</figure>

If you are new to information security but have the passion and desire to dedicate the time that is needed, you will overcome the challenges in front of you and be successful in your quest for the OSCP. I believe that this course is designed in such a way that if a student goes the extra mile, they will be greatly rewarded.

Lets dive in!

# 0x02 - Preparation
Offensive Security has <a href="https://www.offensive-security.com/information-security-training/penetration-testing-with-kali-linux/">stated</a> that students are required to have "A solid understanding of TCP/IP, networking, and reasonable Linux skills". To provide some additional advice, I would suggest looking into the following:

* Using the Linux CLI to perform basic functions such as view and manage file and directory permissions. Understanding how processes and network services are managed. Being able to compile and execute programs.
* Running through the <a href="http://www.offensive-security.com/metasploit-unleashed/Main_Page">Metasploit Unleashed</a> course. If i had to pick two areas in particular, I would say check out exploit development and pivoting.
* Experimenting with different vulnerable machines on <a href="https://www.vulnhub.com/">VulnHub</a> such as Tr0ll, Kioptrix and bWAPP. Also check out <a href="http://overthewire.org/wargames/bandit/">Bandit</a> at OverTheWire to get started.
* Being able to partially read and understand the flow of certain languages such as Ruby, Python, Bash and C. An in depth understanding is not necessary, but try your best to read <a href="http://www.exploit-db.com/exploits/10099/">this</a> and see if it makes sense to you.
* Being interested in information security and having the desire or want to dive deeper into certain subjects.

# 0x03 - Signing Up and Getting Organized
Signing up is a pretty easy process and can be done over <a href="https://www.offensive-security.com/preregistration.php?cid=21">here</a>. On your first day you will receive an email from Offensive Security which contains the following goodies:

* A link for the Kali VM which Off Sec recommends you use.
* Your OffSec credentials.
* The PWK VPN connection info.
* The PWK course guide in form of a PDF and accompanying videos. These resources are absolutely fantastic and will provide you with the fundamentals needed to be successful in the PWK lab.

Before jumping into the lab, I decided to go through all of the course material, videos and exercises. My approach was to:

1. Read a course module while taking notes in the areas which I felt were important
2. Watch the accompanying video for the course module and fill in any additional notes.
3. Complete the exercise for the course module.
4. Rinse and repeat for each module.

I would highly suggest that you leverage a cloud based notebook for the copious amounts of notes that you will be taking. Many people have used KeepNote and synced it with DropBox but I decided to leverage OneNote. This was mainly for the seamless cloud integration, mobile device access, browser access and support across multiple operating systems. If you end up using OneNote, consider enabling the encryption features and password protecting your archive. This is a pretty sanitized view of what my PWK notebook looks like:
<figure>
  <center>
    <img src="https://1.bp.blogspot.com/-Z0D_J0vnhi0/VSU6YhmPzyI/AAAAAAAAB3k/2tAAmgR7yio/s1600/2015-04-08%2B08_25_01-Clipboard.png">
  </center>
</figure>
I cant stress enough the importance of taking notes and screenshots as you go. This has been reiterated to death on many OSCP reviews and should be practiced religiously. In addition, set up regular scheduled snapshots of your Kali virtual machine and keep an encrypted TrueCrypt volume on your virtual host which contains all of the trophies collected from your Kali virtual guest. Try to keep your collection of data inside of your Kali host organized. This is what my container looked like:
<figure>
  <center>
    <img src="https://1.bp.blogspot.com/-Sz6w90ubhbc/VSU7IZuvkzI/AAAAAAAAB3s/_KfuhXq_xGg/s1600/2015-04-08%2B08_28_07-Clipboard.png">
  </center>
</figure>

# 0x04 - The PWK Labs
<figure>
  <center>
    <img src="https://i.imgflip.com/jvtt3.jpg">
  </center>
</figure>

Everyone reading this is going to be in a different stage in life and have a different amount of time that they can dedicate towards the PWK lab. During my time in the lab I was fortunate to have an incredibly supportive girlfriend and a solid group of peers. My schedule basically consisted of:

* Work Mon-Fri from 6:30 AM to 5 PM (including travel time).
* PWK lab 3 or 4 times a week from 6 PM to 10 PM or 11 PM.
* 15-20 hours on the weekend in the PWK lab.

This is what I felt was necessary to ensure success for myself and may not necessarily work for you. It is really important to have a balance between your social life and work/study life. The PWK course is incredibly fun, but at times can be incredibly frustrating. By no means should it take up all of your time. Everyone needs a break, even if its just to take the dog out for a walk or go shoot some pool with some friends!

I want to leave the lab as much as a mystery as possible and not give away any spoilers.

I initially signed up for 90 days and officially started on October 19th 2014. I actually spent about 45 days going through the course material and exercises before even entering the PWK labs.

Once I was coming to the end of the 90 days, I decided to extend my time in the lab for another 60 days. I did this because I wanted to compromise more of the machines in the 2 networks I had gained access to. I also wanted to gain access to the 3rd and final network. Keep in mind, none of this is required, but at the time I felt that if I was able to compromise a majority of the machines and access all of the networks then I would be ready for the exam.

In the first month of being in the lab and around 45 days into the course, I was compromising a new machine almost every single day. My post discovery and enumeration process was basically:

* Penetrate the host.
* Maintain access to the host.
* Conduct thorough post exploitation enumeration on the host.
* Document everything about the host.
* Revert the host.
* Move on to the next host.

But sure enough, all good things must come to an end. The following month was pretty painful, lab machines started getting increasingly difficult to compromise and falling over at the rate of 1 or 2 a week. But through determination, perseverance and repetition of <a href="https://www.youtube.com/watch?v=c3PjAuoTllI">Off Sec's Try Harder song</a>, I was able to compromise most of the other hosts in the other networks.

Halfway through the last month and about 75 days into the labs, I decided that I was ready to take the OSCP exam. I came to this conclusion as I was fairly pleased with my progress through each network and was able to compromise some of the harder machines such as pain, sufferance, freebsd9 and gh0st. Additionally, I would have been left with 15 days of lab access to find out where I went wrong if I were to fail the OSCP challenge the first time around.

This was my own way of measuring my preparation and could be completely different for you. Some students feel ready for the exam after compromising all machines in the public network, other students are happy with compromising 1, 2 or 3 of the other networks. I guess it really comes down to you and how you feel, there is no right or wrong answer :)

Just to quickly chuck this in here, I decided to use automated exploitation through Metasploit as little as possible. Out of all of the machines I compromised, I ended up using Metasploit for less than 10% of them. This was to create a habit of not relying on Metasploit due to the restrictions in the exam which we will get into later. Also, I did not use any vulnerability scanners such as Nessus, Nexpose, OpenVAS etc.

# 0x05 - The OSCP Exam
The OSCP exam is intense. You are provided access for 24 hours to an exam lab which contains machines that you have no previous knowledge of. There are 5 machines which total to 100 points and each machine is weighted differently. To pass the exam you need to gain a minimum of 70 points while adhering to the strict rules of each machine. Some machines will allow for Metasploit access whereas others machines restrict it entirely. You can only use Metasploit on 1 machine and you can only use a specific set of commands.

I started the OSCP exam at 10 AM on March 11th 2015. Overall, the first 12 hours was utilized pretty poorly. I had spent about 2 hours conducting some initial enumeration across all 5 hosts and then 10 hours trying to compromise 3 hosts.

By the time 10 PM came around, I had 45 points and was feeling exhausted. In the following 2 hours I had managed to bump up my points tally to 55, but I could not make any progress on the 2 remaining machines.

I think I lived the definition of insanity between 12 AM to 5:30 AM. I kept on repeating the same old tricks over and over again on the 2 remaining machines, expecting a different result to eventually happen.

At 5:30 AM I threw in the towel and accepted defeat at 55 points. I went to bed feeling incredibly tired and disappointed. The next morning I woke up around 11 AM and still felt pretty down about the whole situation.

The feeling of failure was especially tough on me because throughout my entire academic and professional life I have never failed an exam or certification. Although I didn't realize it at the time, I can truthfully say that failing humbled me. Having the OSCP exam chew me up and spit me back out made me reconsider my entire approach. The first thing few things that came to mind after waking up were:

* Where did I go wrong?
* What can I do to fix it?
* When will I be ready to challenge again?

After a period of self reflection and reviewing some other peoples <a href="http://www.en-lightn.com/?p=941">stories</a>, it all became pretty simple. I made a list of lessons learned that looked similar to the following:

1. I didn't get enough sleep the night before - according to an app I use called "Sleep Time", I had only slept for 5 hours the night before. This was simply not good enough.
2. I didn't take any breaks - I was so focused on getting results that I literally sat in the same chair for 19 hours straight, skipping out on lunch and dinner.
3. I didn't manage my time and rotate between machines - spending 12 hours on 3 machines was pretty stupid in retrospect. I cant even recall why I needed to spend as much time as I did on those 3 boxes. Possibly because I was over-complicating things when I should have been looking at what was staring me directly in the face. I should have utilized my time more efficiently.
4. I had no structure or plan of attack going into the exam - Out of all of the things I have listed, this by far was the biggest point of failure. I tried to wing it through the exam and was punished. My methodological approach needed to approve.
5. Overconfidence is punishable by failure - I went into the exam with the mindset that I had already passed. Don't get me wrong, confidence and the desire to pass is not a bad thing, but being overconfident and thinking you have already passed is something to avoid.

I received my exam results 24 hours after submitting my documentation to Offensive Security. I opened my inbox to find an email which started with with "we regret to inform you..." - I wasn't particularly disappointed at this point as I was expecting this to happen anyways.

To lighten the mood a little, something else that I took out of this experience is that it's totally okay if you don't make the jump the first time. Learn from the experience and find out how you can fix where you went wrong.

<figure>
  <center>
    <img src="https://i.imgur.com/sQV6A0P.gif">
  </center>
</figure>

I decided that I was going to spend 3 weeks trying to work on the areas which needed some attention. I was confident with my technical penetration testing skills but I knew that my approach and time management needed some evaluation.

I started reading methodologies created by different communities or people such as OWASP, OSSTMM, 0DaySecurity.com and VulnerabilityAssessment.co.uk.

I then created my own methodologies that consisted of a holistic penetration testing approach leveraging manual testing techniques in the following areas:

1. Host Discovery
2. Enumeration and Service Discovery
3. Vulnerability Assessment and Analysis
4. Web Application Penetration Testing
5. Infrastructure Penetration and Privilege Escalation
6. Post Exploitation

I then tried, tested and refined these methodologies in the PWK lab and against several vulnerable machines on VulnHub.

After creating some solid methodologies I felt that I was ready to give the OSCP exam another shot and so I scheduled the exam at 9 AM on April 3rd 2015.

<figure>
  <center>
    <img src="https://lh3.googleusercontent.com/proxy/5pFd0Mmn81UxiexqhTtTjCwSvhib9K-OmN0p-7-nWh58YGUba3S32u4Yq6IDrf8il3U=s0-d">
  </center>
</figure>

I spent the day before the exam resolving another lingering issue, this being time management. I decided to create some automated alerts in my calendar to help remind me to rotate boxes and take breaks. I didn't bother taking a screenshot past 9 PM, but you can get an idea of what the first 12 hours looked like:

<figure>
  <center>
    <img src="https://3.bp.blogspot.com/-5yOX15RBn-k/VSVab4DGY8I/AAAAAAAAB38/IWy_wRt209c/s1600/sc.png">
  </center>
</figure>

I also went to bed at 11 PM the day before the exam and woke up around 8 AM. I decided to head out and grab a coffee and some breakfast and got back home at 8:45 AM feeling sharp and prepared for the upcoming challenge.

Before jumping into the OSCP challenge, something else I changed this time around was playing music throughout the exam. During the first sitting I practically sat in silence for 19 hours. But this time around, I had some tunes playing in the background. Mostly electronic music from artists such as Pendulum, Netsky, Deadmau5 and Knife Party.

During the second attempt, I was lucky to receive 1 machine which was on the previous exam. I had invested a lot of time in the concepts needed on this particular machine (cough.. BOF.. cough..) on the previous exam so it was totally okay to be greeted by an old friend.

The other 4 machines were completely new and probably harder than the machines I received the last time around. I promised myself that I would leverage my new methodologies throughout the entirety of the exam. Especially during everything pre-penetration. And boy am I glad that I did.

I spent the first 3 hours (9 AM - 12 PM) of the exam enumerating everything. I then dived deeper into the things I enumerated and enumerated some more. This strayed away from my schedule, but I wasn't too worried because I was still on track. I broke for lunch at 12 PM and watched some cat videos on YouTube.

3 hours after all of my enumeration was complete (3:30 PM), I had full root/SYSTEM access on 3 machines which put me at 65 points. More importantly, I completely understood exactly what needed to be done on 1 of the 2 remaining hosts, but this would require leveraging the trick card in my back pocket; Metasploit. And so I decided to put that off for as long as possible. The methodological approach I had created in the previous weeks was 100% responsible for my early success.

I switched my attention to the host which I was unfamiliar with and spent the next 5 hours trying to get access to that machine. I came very, very close but I had decided that I should stop pursuing it and let it go.

I went back to the host which I could use Metasploit on and carefully reviewed my enumeration to ensure that the exploit would work. I'm sure that you are familiar with the saying "measure twice, cut once", I think I measured 10 times. After some serious contemplation, I took a deep breath, hit the enter key and fired off my exploit. BOOM! I was at 75 points, exceeding the minimum requirements set by Offensive Security to pass OSCP challenge. Queue the root dance:

<figure>
  <center>
    <img src="https://lh4.googleusercontent.com/proxy/yTFOJafw-UnwxRoAw4Lu1fiG3ghnpBa8qnGvgWl32jcuThKEfd2V7ikkwMsPczsFH9ul4qFOSgvUMqIGij7rPVFMGC27SjdZvYfEb2GItaVv=s0-d">
  </center>
</figure>

Within 12 hours I had 75 points and full root/SYSTEM access on 4 machines. To put this in perspective, on my first attempt at the exam I was sitting at 45 points in the first 12 hours. Again, the heavy focus on enumeration paired with the methodological approach I had taken was 100% responsible for my success.

My root dance was interrupted abruptly by the reminder that I still had to compose and submit my documentation. Feeling pretty pleased with myself, I started working on the documentation that night. I worked on the title page, executive summary and overall structure of the template for a few hours before going to bed.

I decided to sleep in the next morning and started working on the documentation around 11 AM. I really wanted to create a quality document for Offensive Security to read and so I invested about 10 hours writing it. The document also included a "vulnerability analysis" section which outlined the issues present with the host I was unable to compromise. It also had recommendations to remediate the issues identified with the uncompomised host. At the time of completion it was 130 pages and had been QA'd twice.

I submitted my documentation to Offensive Security at 9 PM on April 4th 2015. This was 36 hours after I had started the exam. The sense of relief I felt after submitting the document was absolutely magnificent. It was the culmination of 6 months of hard work being expressed in a fully practical exam with the main deliverable being a formal penetration test report.

On April 6th 2015 at 6 AM I received the email I had been obsessing over since submitting the documentation. Confirmation of my success in completing the OSCP exam and obtaining the OSCP certification.

# 0x06 - Other Considerations
Employers should know that the OSCP is an incredibly hard certificate to achieve. Offensive Security has <a href="https://www.offensive-security.com/information-security-training/penetration-testing-with-kali-linux/">stated</a> that people who have obtained the OSCP are able to defeat any learning plateau and have a complete understanding of a penetration test from start to finish.

Something I read and really liked on <a href="http://www.en-lightn.com/?p=941">Nick Schroedl's</a> blog is:

*"The OSCP is one of a handful of certifications that is achieved through hands on scenario testing rather than multiple choice. You can guess a, b, c, or d and have a %25 of getting it right. [In] lab scenarios there is no guessing."*

I also think that the exposure of the OSCP in industry is expanding. I recently read the new PCI Penetration Testing Guidance Report (March 2015) and I was really happy to see that under section 3.1 the OSCP is at the top of the list. In previous years this may not have been the case.

The amount of respect that I have gained for fellow OSCP holders is incredible. Not only for their clear demonstration of practical penetration testing, but also for their dedication and determination to try harder!

# 0x07 - Shoutouts
I wanted to give a shout out to my fantastic girlfriend who provided me with continuous reassurance, support and advice over my time in the course. Even though she may not have fully understood the intricate technical details of what I was doing, she understood how much the OSCP meant to me and supported me in every possible way to achieve my goals.

Also a massive thank you to both <a href="https://twitter.com/retBandit">Chis Thompson</a> and <a href="https://twitter.com/alastairgray">Alastair Gray</a> for putting up with my continuous pestering and complaining. Chris and Alastair both have OSCP's and were sort of like role models for me. Good help is hard to come by, this is especially true in info sec. These two awesome individuals were happy to put up with me when they could have easily told me to... yeah.

So whats next?

I'm going to be taking a few weeks off and then probably start studying for the CEH, after that I'll be looking into the GIAC GPEN and CISSP. Eventually making my way to the OSCE.

If you made it this far, thank you for taking the time to read my review of the PWK Course and OSCP Exam. Feel free to contact me if you have any questions!
