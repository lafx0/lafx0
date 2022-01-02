---
title: CRTP - Attacking and Defending Active Directory ( just another review )
date: 2021-03-08
description: CRTP certification review
tags: ["crtp", "active directory", "review", "security", "hacking"]
---

I recently completed the "[Attacking and Defending Active Directory](https://www.pentesteracademy.com/activedirectorylab)" course from Pentester Academy, and successfully obtained the CRTP certification. This will be my personal review of the course, including some advices that could help those that are thinking about signing up for the training.

---

### Table of contents

- [Intro](#intro)
- [Course materials](#course)
- [Labs](#labs)
- [Exam](#exam)
- [Tips](#tips)

---

## Intro {#intro}
My background mostly resides in web programming and since I started working in security full-time I mainly focused on Web Application hacking. I started gaining interest in Active Directory and Windows after a long period of research and preparation for the eWPTX certification - which is all about advanced Web Application exploiting techniques - and wanted to concentrate on something that was out of my "comfort zone".
I slowly started getting involved in more internal pentests at work and decided to focus my energies in expanding my knowledge about AD and Red Teaming.

CRTP is a very beginner friendly course and for many could be too basic. I decided to go for it (and not directly for CRTE) because I wanted to make sure to master the basics before focusing on more advanced topics.


## Course materials {#course}
The course materials consists of a series of videos and slides focused on explaining what Active Directory is and what are some common misconfigurations that one can encounter during an assessment. The exposition is in my opinion very clear and concise and every topic is followed by a practical hands-on where the author - Nikhil Mittal - shows how to exploit the presented misconfiguration.  

Here is a list of topics that the course covers:
- Domain Enumeration
- Local Privilege Escalation (briefly)
- Lateral Movement
- Domain Persistence
- Domain Privilege Escalation
- Cross-Forest Attacks
- Forest Persistence
- Detection and Defense

The section about "Detection and Defense" is not the main selling point of the course, but in general it gives some good recommendations on how to remediate the most common AD misconfigurations.  

## Labs {#labs}
![CRTP_lab](/img/activedirectorylab.png)

The labs consists of a two-forest environment containing different machines. In my opinion this is the main selling point of the course. They are very stable and can be accessed via VPN or via an online RDP access. This is where you will sharpen your enumeration skills and perform all the exploitation techniques that will grant you Enterprise Admin access in the environment.

I solved the labs multiple times, using different tools and trying to really understand what was going on under the hood. Once you identify the main attack path the labs can be solved in few hours, but here my advice is to slow down and take some time in order to become familiar with the tools and its outputs.
The true value of the labs lies in how deeply one is willing to go to understand the covered topics.


## Exam {#exam}
The exam environment is very similar to the one available in the labs. The student is required to exploit some AD misconfigurations in order to obtain RCE from five machines and to produce a detailed report containing the steps followed to compromise the environment.
I felt very confident before my exam attempt as I made sure to understand very well every topic presented in the course materials and solved the labs several times and with different techniques. But I was wrong and failed my first attempt, as I only compromised two machines and get stuck at compromising the third. I had some problems with getting consistent results with the tools used during the labs and also omitted a few important steps during my initial enumeration.

The CRTP exam has a 30 days cool down period, so immediately after the exam attempt I signed for the CRTE Advanced Bootcamp edition. I was pretty confident about my skills and really wanted to continue my studies on Active Directory. I used the new labs as a proving ground for new and advanced techniques, reinforcing the knowledge I got from the CRTP course.

I recently attempted the exam again and this time I was able to compromise all the five machines in the environment and obtain the CRTP certification. I have to admit I had a hard time finding out the attack path for the third machine due to missing informations, so an important lesson that I have learned is to always enumerate everything and multiple times when without ideas on how to proceed.

## Tips {#tips}
I recommend the course to anyone who wants to learn the basics on how AD could be abused from an offensive side. It is not an advanced course, so if you are already familiar with most topics of the course content, CRTE could be a better fit.

The following are my tips that will hopefully help in achieving the CRTP certification.

- __Go beyond the course materials:__ Do not limit yourself on studying on the course materials only. Make sure to research the topics that you find interesting and - more importantly - that you find hard to understand. There are plenty of blog posts out there that will explain things in different ways. Some of them will be more congenial to you. I found great value here: http://blog.harmj0y.net
- __Make sure to understand the concepts:__ Make sure to understand the concepts very well in theory before going straight to the exploitation phase. This is important because there will be different scenarios where the same commands will not work and you will need to make some slight modifications based on the particular case.
- __Create a command reference:__ A lot of the commands presented in the course materials will be used continuously during the labs and the exam. Make sure to create a simple list of all the commands to use as a reference, in order to save some time searching for them every time in the slides!
- __Treat the exam as every other security assessment:__ Do not suppose that the exam is based solely on the topics presented in the course materials. Be sure to treat the exam as any other security test and make sure to enumerate everything!  
