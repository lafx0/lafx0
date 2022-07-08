---
title: CRTP - Attacking and Defending Active Directory ( just another review )
date: 2021-03-08
description: CRTP certification review
tags: ["crtp", "active directory", "pentester academy", "certification", "review"]
categories: ["Active Directory", "Red Team"]
summary: Pentester Academy CRTP - course and exam review.
---

I recently completed the "[Attacking and Defending Active Directory](https://www.pentesteracademy.com/activedirectorylab)" course from Pentester Academy, and successfully obtained the CRTP certification. This will be my personal review of the course, including some tips that could be helpful to those in the middle of the training.


---

## Intro {#intro}
My background mostly resides in web programming and since I started working in security full-time I mainly focused on Web Application hacking. I started gaining interest in Active Directory and Windows right after a long period of research and preparation for the _eWPTX_ certification - which is all about advanced Web Application exploiting techniques - and wanted to concentrate on something that was slightly different.
I slowly started getting involved in more internal pentests at work and decided to focus my energies in expanding my knowledge about AD and Red Teaming.

CRTP is a very beginner friendly course and for many could be too basic. At first I wanted to go straight for CRTE and give-up on some basic topics, but then I decided to go for it as I wanted to make sure to master the basics before focusing on more advanced topics.


## Course materials {#course}
The course materials consists of a series of videos (up to 14 hours) and slides aimed at explaining what Active Directory is and what are some common misconfigurations that one can encounter during an assessment. The exposition is in my opinion very clear and concise and every topic is followed by a practical hands-on where the well known instructor - [Nikhil Mittal](https://twitter.com/nikhil_mitt) - shows how to exploit the presented misconfiguration.  

The following are the topics covered in the course:
- Domain Enumeration
- Local Privilege Escalation (briefly)
- Lateral Movement
- Domain Persistence
- Domain Privilege Escalation
- Cross-Forest Attacks
- Forest Persistence
- Detection and Defense (briefly)

The "_Detection and Defense_" section is not the main selling point of the course, but in general it gives some good recommendations on how to remediate the most common AD misconfigurations.  

## Labs {#labs}
![CRTP_lab](/img/activedirectorylab.png)

The labs consists of a two-forest environment containing various machines. In my opinion the labs are the main selling point of the course. This is where you will sharpen your enumeration skills and perform all the exploitation techniques that will grant Enterprise Admin access in the environment.
The labs are very stable and can be easily accessed via VPN or via RDP over Guacamole.

I solved the labs multiple times using different tools and focusing to really understand what was going on under the hood. Once you get familiar with the environment and identify the main attack path, the labs can be easily solved in a few hours, but here my advice is to slow down, to spend some time on sharpening your enumeration skills, trying out different tools and reviewing its outputs.


## Exam {#exam}
The exam environment is very similar to the one available in the labs. The student is required to exploit some AD misconfigurations in order to obtain RCE from five machines and to produce a detailed report containing the steps followed to compromise the environment.

I felt very confident before my exam attempt as I made sure to understand very well the theory presented in the course materials and solved the labs several times using different techniques. But I was wrong and failed my first attempt, as I only compromised two machines and get stuck at the third. I had some problems getting consistent results with the tools I used during the labs and also omitted a few important steps during my initial enumeration.

The CRTP exam has a 30 days cool-down period, so immediately after the first exam attempt I signed up for the _CRTE Advanced Bootcamp Edition_. I was pretty confident about my skills and really wanted to deepen my knowledge on Active Directory. I used the CRTE bootcamp lab environment as a proving ground for learning new advanced techniques, while reinforcing the knowledge I got from the CRTP course.

I recently attempted the exam again and this time I was able to compromise all the five machines and to obtain the CRTP certification. I have to admit I had a hard time finding out the attack path for the third machine (again), and an important lesson I have learned one more time is the usual one - _enumerate, enumerate, enumerate!_

## Tips {#tips}
I recommend the course to anyone who wants to learn the basics on how to attack Active Directory. Keep in mind that this is not an advanced course, so if you are already familiar with most of the covered topics, CRTE could be a better fit for you!

The following are my tips that will hopefully help other students in achieving the CRTP certification.

- __Go beyond the course materials:__ Do not limit yourself on studying on the course materials only. Make sure to research the topics that you find interesting and - more importantly - the ones that you find hard to understand. There are plenty of blog posts out there that will explain things in different ways. Some of them will be more congenial to you. I found great value here: http://blog.harmj0y.net
- __Make sure to really grasp the concepts:__ This is important because in the labs some scenarios will feel contrived and will work just for showcasing the intended exploitation. I think that the real benefit of the course resides in learning concepts in details, in order to be able to reuse them in real-life work situations.
- __Create a command reference:__ A lot of the commands presented in the course materials will be used continuously during the labs and the exam. Make sure to create a simple list of all the commands to use as a reference, in order to save time searching for them every time in the slides!
- __Treat the exam as every other security assessment:__ Do not suppose that the exam is based solely on the topics presented in the course materials. Be sure to treat the exam as any other security test and enumerate everything!

---
Good luck with your CRTP exam and feel free to reach out on Twitter for further questions!
