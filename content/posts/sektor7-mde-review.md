---
title: Sektor7 Red Team Operator - Malware Development Essentials Course review
date: 2022-02-10
description: Sektor7 Red Team Operator - Malware Development Essentials Course review
tags: ["sektor7", "malware development", "review", "red teaming"]
categories: ["Red Team", "Malware Development"]
summary: My review of the Sektor7's MDE  course.
---

![Sektor7_MDE](/img/sektor7_mde.png)

As penetration testers / ethical hackers we are constantly relying on open source or commercial tools to perform different tasks. A quick Google search on these tools shows plenty of articles and blogs on how to use them, but very few focus on how they work and behave internally. 

When I started my offensive path a few years ago I remember the amazement I felt when using tools like Empire or Metasploit for the first time. I've always been fascinated by the idea of developing offensive tools instead of just using them blindly and when I first discovered Sektor7's training, it didn't take me long to sign up for the Malware Development Essentials course. 

If you are curiuos too about how these offensive tools are working under the hood, this course is probably for you! 

---


## Course content {#course-content}

The [Malware Development Essentials](https://institute.sektor7.net/red-team-operator-malware-development-essentials) course from <u>Sektor7</u> is an introductory course for pentesters / red teamers and blue teamers wanting to learn the basics of offensive malware. 

As the course page states:

{{< lead >}}
"It will teach you how to develop your own custom malware for latest Microsoft Windows 10. And by custom malware we mean building a dropper for any payload you want (Metasploit meterpreter, Empire or Cobalt Strike beacons, etc.), injecting your shellcodes into remote processes, creating trojan horses (backdooring existing software) and bypassing Windows Defender AV."
{{< /lead >}}


In particular, the course sections are:

- Introduction to Malware Development
- Format and structure of PE files
- Payload encoding and encryption
- Function call obfuscation
- Backdooring existing software
- Code Injection into remote processes


All the content is available via an online platform that hosts multiple videos for each section of the course. 
The student also receives a Windows 10 pre-configured VM with a working environment for developing and testing the code templates used in the videos. 

Note that <u>the course is not associated with a certification exam</u> - which I think is a very good choice (the infosec certification drama is becoming embarrassing, don't you think too?!). 
Instead, as last requirement, the student is asked to code a custom dropper that needs to meet some stealthy requirements.   


## What I learned from the course {#what-i-learned}

I really enjoyed everything from the course, but I tried to get the most out of the <i>Function Call Obfuscation</i> and <i>Remote Process Injection</i> topics. I had some general understanding of those subjects, but seeing a practical implementation explained by an excellent instructor helped a lot in clearing away some doubts. 

For every code template I encountered, I spent a good amount of time on MSDN researching about every function and Windows API, on what it was doing, accepted parameters, return values, types, etc. 
Not only my general understanding of Windows APIs improved, but also my C programming skills have benefited. Very soon I found myself to be much more confident with the code and able to easily adapt it to my own needs. 

For the Encryption section, I ended up coding a tool very similar to the one provided in the materials that is able to AES / XOR encrypt strings and files. I ported the original code to Python3 and added some enhancements that I found useful for my approach.

If you wanto to give it a look, here it is --> 
[AES/XOR Encryptor](https://github.com/lafx0/AESXOR-Encryptor)

I learned the most from the final assignments, where the student needs to come up with his own solutions as they are not provided.

This forced me to focus on adapting the code to my own solutions (may release them later) instead of blindly relying on the provided templates, and researching functions and APIs that were not covered in the course materials.

At the end I came up with a working custom dropper supporting AES for string and function calls encryption and I learned how to hide encrypted payloads behind images and separate encrypted files. Every now and then I'm still thinking about different ways of solving the assigments and trying to implement different ideas. 

Somehow I feel like I got more from this small course than from other ones that were made of hundreds of slides. 


## My thoughts on the course {#my-thoughts}

I found the videos to be very instructive; all the topics are explained in a clear and concise way that is impossible to not undestand the topic. 

People who are not familiar with C/C++ (or programming in general) may have some difficulty in following along with the material and they may need some more time to understand what's going on with the code. But that's ok, one can learn everything at his own pace. The topics are wisely built one on top of each other and it is not difficult to grasp it with constant practice. 

Luckily I already had some experience with C++ from university which really helped in following along and allowed me to focus on the concepts behind the code.  

My advice is not to rush through the course videos and materials with the aim of getting the certificate of completion in order to put it on LinkedIn to look cool.

The course, compared to others, is somehow short and can be completed probably in 2/3 days but I don't think this was the goal of the author. You could just take a look at the code samples, compile it and proceed to the next chapter. Or you can play with the code and take some time to improve and appreciate the process. Your choice. 

The real value of the course resides in my opinion in the time you are willing to put in and in the knowledge that you will gain while patiently working on the material. 


## Conclusion {#conclusion}

My experience with the course was more than positive. I do recommend Sektor7's Malware Development course because it really stands out in the infosec panorama. 

I took several "high priced" courses and this one   is simply on another level when talking about content clarity and gained practical knowledge. 

This is why I will definetely continue my journey with the Intermediate course!

---


That was it! 

Feel free to connect on [Twitter](https://twitter.com/_lafx0)







