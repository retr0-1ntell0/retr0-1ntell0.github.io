---
title: If You Do It Twice, Automate It
date: 2026-01-27 10:10:10 -5000
categories: [HTB, customization, Linux]
tags: [Automation, Hacking, Workflow] 
toc: true
comments: false
pin: true
description: Repetition slows you down, especially during reconnaissance. This post shows how I automated my everyday hacking workflow using `.zshrc`. The goal is simple, stop typing the same commands and start hacking faster.
---


![Image-description](/Assets/Pictures/bruce-almighty.gif)

## Introduction

It all started with my journey as a hacker on **Hack The Box**. At first, it was about learning — breaking machines, understanding vulnerabilities, and getting comfortable with the process. Over time, I moved into more competitive hacking, where speed actually mattered. Machines were being pwned in record time, and rankings depended on how fast you could get it done.

**That phase taught me something important: speed is a skill, not just talent. And more importantly, it exposed a flaw in my workflow**.

I realized I was doing the same repetitive tasks over and over again — creating the same directories, running the same scans, setting up the same listeners — every single time I started a new machine. That’s when this quote came back to mind:

> “Insanity is doing the same thing over and over again and expecting different results.”
>
> — often attributed to Einstein (attribution disputed)
{: .prompt-info }


If I wanted better results, my reconnaissance process needed to change. Not by adding more tools or writing massive scripts, but by removing friction from the basics. Instead of building full scripts, I started experimenting with my `.zshrc` file — turning repeated actions into simple aliases and functions that matched the way I actually work.

This post is about that mindset shift: optimizing the boring parts so I can focus on what actually matters.

## Set up

First, you need a running **Linux system**… duh.

This works with both `.zshrc` and `.bashrc`, since we’ll be using aliases and basic shell functions. Whether you’re on a virtual machine or bare metal, everything here should work the same.

That’s really all you need: *a working Linux environment*. Simple, right?


## The real deal

Here is **my workflow**. I first need to create some folders, that will help organize my work. Here are the file structure I use:

#### For simple machines or challenges

```bash
~/HTB-Machine                                                                                      
> tree . 
.
├── Evidence
│   ├── creds
│   ├── data
│   └── screens
├── Logs
├── Scans
└── Tools

8 directories, 0 files
```
{: .nolineno}

That's *8 directories* that I always use when it comes to simple machines. So every time I will have to create them in order for me to store all the necessary files. Now imagine doing this every single time using `mkdir`...that's insanity.

#### For Prolabs, Certifications or Penetration Tests

For `Prolabs` or `Pentest` the file structure is also different. 
As shown below:

```bash
~/HTB-ProLab  
> tree .
.
├── External
│   ├── evidence
│   │   ├── creds
│   │   ├── data
│   │   └── screens
│   ├── logs
│   ├── scans
│   ├── scope
│   └── tools
└── Internal
    ├── evidence
    │   ├── creds
    │   ├── data
    │   └── screens
    ├── logs
    ├── scans
    ├── scope
    └── tools

19 directories, 0 files
```
{: .nolineno}

Having an automated way to create these folders every time turned out to be a really good practice. It helped me save time on some basic tasks such as directory creation, nmap scans, replacing the IP of the machine you are working on, verifying your HTB IP, even setting up a listener. The list goes on....

This means that every time I need to create directories or run repetitive tasks like `nmap` scans, I can do it with a single command. No need to remember the file structure or retype commands — if I’ve done it enough times, it gets automated. I am not insane. Now that you catch my drift, let's go through with weaponization of our Linux system. 


## Weaponization

### Aliases

Let's start with the aliases I found useful:

#### OpenVPN stuff

I have a directory where all my vpns are stored `~/Downloads/vpns/`

```bash
alias conn="cd ~/Downloads/vpns/"  # gets you in the vpn directory
alias vpn="sudo openvpn"           # Help starts vpn => $ vpn labs.ovpn
```
{: .nolineno}

#### SSH stuff
This is for personal preference. I rather have **SSH** not read or write on my `known_hosts` file and remove the interactive prompt.

```bash
alias SSH="ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null "
alias SCP="scp -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null "
```
{: .nolineno}

Together, these options mean:

`StrictHostKeyChecking=no`

- Automatically accepts the host key

- No interactive prompt

`UserKnownHostsFile=/dev/null`

- Doesn’t read or write ~/.ssh/known_hosts

- Host keys are forgotten immediately

👉 **Result**: *non-interactive, stateless SSH*

**TL;DR**

- ✅ Correct for HTB / CTF / labs
- ❌ Unsafe for real infrastructure (that's why they are capitalized in my `.zshrc`)


#### Checksec

I use **pwntools** and their new output is not my favorite. I liked the old one better so here is the workaround I did.

```bash
alias checksec="pwn checksec"    # just for better output
```
{: .nolineno}


#### Directories
This helps me move quickly in my file system. one command and I am where I wanted to be.

```bash
alias lab="cd $HOME/HTB/Machines"    # Going straight to the machine folder

alias chal="cd $HOME/HTB/Challenges"    # Going straight to the machine folder

alias prolabs="cd $HOME/HTB/ProLabs"    # Going straight to the machine folder

alias ctfs="cd $HOME/CTFs"    # Going straight to the machine folder

alias fort="cd $HOME/HTB/Fortress"    # Going straight to the machine folder

alias www="cd /run/media/$USER/ssd/www/"    # I use my ssd to store the tools I can upload to the victim  

alias bugb="cd $HOME/BugBounty/"    # Going straight to the Bug Bounty folder

# These two aliases are used in some functions 
alias hax="mkdir -p Evidence/{creds,data,screens} Logs Scans Tools && echo -e '\033[1;32mDirectories Created !\033[0m'"    # This helps me creating all the directories I will need to work on a machine

alias pentest="mkdir -p {External,Internal}/evidence/{creds,data,screens} {External,Internal}/logs {External,Internal}/scans {External,Internal}/scope {External,Internal}/tools && echo -e '\033[1;32mDirec
tories created !\033[0m'"    # This helps me creating all the directories I will need to work on a Prolab or real penetration test

```
{: .nolineno}

#### HTB IP
This one is just because `ip` commands or `ifconfig` are too verbose for me to just copy paste the HTB ip at times. I am that lazy. 

For this you have to make sure `ip` command works on your host, this was crafted specifically for the output of `ip -br a`

```bash
alias htbip="ip -br a | grep tun | awk '{print \$3}' | sed 's#/.*##'"
```
{: .nolineno}

In case you do not have this package installed, you can install per Linux distribution as shown below:

🐧 **Debian / Ubuntu / Kali / Parrot**
```bash
sudo apt update
sudo apt install iproute2
```
{: .nolineno}

🎩 **RHEL / CentOS / Rocky / Alma / Oracle Linux**
```bash
sudo dnf install iproute
```
{: .nolineno}


(Older systems may use yum instead of dnf)

🧢 **Fedora**
```bash
sudo dnf install iproute
``` 
{: .nolineno}


🦎 **openSUSE**
```bash
sudo zypper install iproute2
```
{: .nolineno}


🐧 **Arch Linux / Manjaro**
```bash
sudo pacman -S iproute2
```
{: .nolineno}


🐧 **Alpine Linux**
```bash
sudo apk add iproute2
```
{: .nolineno}


🧪 **Gentoo**
```bash
sudo emerge --ask sys-apps/iproute2
```
{: .nolineno}


#### Nmap commands
So this is the most insane part. you can run these commands even asleep in your head at this point. Why keep typing it ? How insane one has to be ? You are losing few seconds or minutes typing all of this every time you have to start an engagement.

> Here make sure that you have an environment variable with the right value as IP. Be aware and always make sure with `echo $IP`
{: .prompt-tip }

```bash
alias openPorts='nmap -Pn -p- -vv -T4 --open -oN ./all-open.out $IP'    # Scan for all open ports

alias openFastPorts='nmap -Pn -F -vv -T4 --open -oN ./fast-port-scan.out $IP'    # Scan for ~100 most common ports

alias serviceScanAll='sudo nmap -Pn -sV -sC -p- -vv -T4 -oN ./services-all-scan.out $IP'    # Service scan for all the open ports

alias serviceScanFast='sudo nmap -Pn -sV -sC -vv -T4 -oN ./services-fast-scan.out $IP'    # Service scan for ~100 most common ports

alias UDPScanALL='sudo nmap -Pn -sU -p- -vv -T4 -oN ./all-open-UDP.out $IP'    # Regular UDP all ports scan

alias UDPScan='sudo nmap -Pn -sU --open -vv -T4 -oN ./open-UDP.out $IP'    # Regular UDP open ports scan

```
{: .nolineno}

> P.S: Aliases don't need parameters and can just be run as any other commands. So please for these `nmap` make sure the IP is the correct one.
{: .prompt-warning }

### Bash functions

This is the fun part. Instead of having scripts, why not make functions in our `.zshrc` or `.bashrc` file instead ?
I have free will so I did the following.

#### Helps create a new folder for Machines
This function here uses an alias we have called `hax` that will help not only create the new folder but also add all the necessary folder you will need for documentation as mentioned earlier in the [aliases](#directories). 


```bash
function box() {
    if [ -z "$1" ];then
        echo "\033[1;31mProvide a Machine name !\033[0m"
        return 1
    fi
    lab && mkdir -- "$1" && cd "$1" && hax;
}
```
{: .nolineno}

This function takes 1 parameter which is the name of the machine/box:

eg:
```bash
box Testos
```
{: .nolineno}

#### Starting a reverse listener

```bash
function revListener() {
    if [ -z "$1" ];then
        echo "\033[1;31mProvide the listening port !\033[0m"
        return 1
    fi
    rlwrap -car nc -nlvp $1
}
```
{: .nolineno}

This function takes one argument which is the port to listen to:

eg: 
```bash
revListener 9009
```
{: .nolineno}


#### Helps create a new folder for Prolabs/Pentest
This function here uses an alias we have called `pentest` that will help not only create the new folder but also add all the necessary folder you will need for documentation as mentioned earlier in the aliases.

```bash
function pro() {
    if [ -z "$1" ];then
        echo "\033[1;31mProvide a Lab name !\033[0m"
        return 1
    fi
    prolab && mkdir -- "$1" && cd "$1" && pentest;
}
```
{: .nolineno}

This function takes 1 parameter, which is the name of Lab/Pentest.

eg: 
```bash
pro ACME.inc
```
{: .nolineno}

#### Function to change the value of $IP

Each engagment has a different IP, so changing the environment variable $IP is important for me since I use that variable for all my `nmap` scans as shown @TODO

Just make sure your `.zshrc` or `.bashrc` contain a line like this
```bash
export IP=0.0.0.0      # IP variable placeholder
```
{: .nolineno}

This will help you see if your variable is set correctly. If you run `echo $IP` and it shows `0.0.0.0`, then you know you did set your IP variable yet.


```bash
function replaceIP() {
    if [ -z "$1" ]; then
        echo "\033[1;31mProvide a new IP!\033[0m"
        return 1
    fi
    sed -i "s/^export IP=[^ ]*/export IP=$1/" ~/.zshrc
    echo "IP address updated to $1"
    source ~/.zshrc
}
```
{: .nolineno}

This takes 1 argument which is the IP to replace to our default value of `0.0.0.0`

eg: 
```bash
replaceIP 10.10.14.14
```
{: .nolineno}

#### Adding an host to the `/etc/hosts` file

This comes handy when you need to append a new host to your `/etc/hosts` file. This is how quickly I am doing it now. 

```bash
function hostAdd(){
    if [ $# -ne 2 ];then
        echo "\033[1;31mProvide the I.P and hostname to add !\033[0m"
        return 1
    fi
    echo -e "$1\t\t$2" | sudo tee -a /etc/hosts
}
```
{: .nolineno}

It takes 2 argumentts which are the IP and the hostname.

eg: 
```bash
hostAdd 10.0.0.12 testos.lolz
```
{: .nolineno}

#### Tranfering tools to current working directory

I have a folder with tools that I can transfer such as `Rubeus.exe`, `nc.exe`, `RunasCs.exe`...just to name a few. That file is called `www` (ippsec way ;) )
So I made this function to just help me tranfering files with one command knowing the file I want to transfer.

```bash
function wwwToVictim(){
    if [ $# -ne 1 ]; then
        echo "\033[1;31mWhat is the filename you need to add to the current directory ?\033[0m"
        return 1
    fi
    echo "\033[1;32mCopying file $1 to $(pwd)\033[0m"
    cp -v "/run/media/$USER/ssd/www/"$1 $(pwd)
    echo "\033[1;32m[+] Done\033[0m"

}
```
{: .nolineno}

This function only takes one argument which is the file we want to copy. I made this function because in my workflow, I've got a folder named `tool` where I store all the downloaded or uploaded tools which hacking/pentesting. That's why it takes one argument and copy it to the current working directory. It can be ran as shown below:

eg:

```bash
wwwToVictim Rubeus.exe
```
{: .nolineno}

This will copy the file `Rubeus.exe` to the current directory.

#### Spinning Python server dynamically
Contraty to the alias, here we are spinning the webserver dynamically. This came in handy when port 80 was busy. I needed a dynamic way to spin the server with any random port.

```bash
py_serve() {
    local PORT=${1:-8000}

    if ! [[ "$PORT" =~ ^[0-9]+$ ]] || [ "$PORT" -lt 1 ] || [ "$PORT" -gt 65535 ]; then
        echo "[-] Invalid port: $PORT"
        echo "Usage: py-serve [port]"
        return 1
    fi

    if [ "$PORT" -lt 1024 ]; then
        echo "[*] Starting Python HTTP server on privileged port $PORT (sudo)"
        sudo python3 -m http.server "$PORT"
    else
        echo "[*] Starting Python HTTP server on port $PORT"
        python3 -m http.server "$PORT"
    fi
}
```
{: .nolineno}

This function takes only 1 or 0 argument, which is the port python would be opening to server. when no argument is set, it defaults to port `8000`.

> Just a reminder that ports below 1024 require sudo.
{: .prompt-tip}

Unless you want to grant python permission to bind to low ports (Ports < 1024). (Which I won't recommend)
You can do the following to set it up:
```bash
sudo setcap 'cap_net_bind_service=+ep' /usr/bin/python3
```
{: .nolineno}

Once done we can verify it with:
```bash
getcap /usr/bin/python3
```
{: .nolineno}

If you can do it, you can also undo it:
```bash
sudo setcap -r /usr/bin/python3
```
{: .nolineno}

eg:
```bash
py_serve         # default port
py_serve 8081    # starting the server on 8081 
```
{: .nolineno}


## Key Takeaways

#### TL;DR

- Speed matters in hacking, especially in competitive environments.
- Repeating the same setup tasks manually is wasted time and mental energy.
- If you do something often enough, it should be automated.
- Shell customization (`.zshrc` / `.bashrc`) is a lightweight, flexible alternative to full scripts.
- Optimizing your workflow lets you focus on what actually matters: hacking.



There is also a tool I recommend for recon with bugbounty as I contributed a little on my friend's [Durgaprasad-123](https://github.com/Durgaprasad-123/Personalrecon/commits?author=Durgaprasad-123)'s project can be found [here](https://github.com/Durgaprasad-123/Personalrecon).

I also made a repository, where you can just go and copy paste all of these details. That file would be updated depending on my needs or suggestions. But the article might not be. [Here](https://github.com/retr0-1ntell0/ZSH-Recon-Workflow.git) is the link to the repository.

## Conclusion

At the end of the day, this isn’t about being lazy — it’s about being efficient. When you’re hacking, your brain should be focused on understanding the target, chaining vulnerabilities, and thinking creatively — not remembering directory structures or retyping the same commands for the hundredth time.

Customizing your shell is one of those low-effort, high-reward improvements that quietly levels you up. Every second you save on setup is a second you can spend actually hacking. Over time, that adds up.

You don’t need my exact aliases or functions to benefit from this approach. The real takeaway is this: pay attention to what you repeat. If you catch yourself doing the same thing more than a few times, automate it — whether that’s with aliases, functions, or small scripts that fit your workflow.

Tools don’t make a hacker better. Habits do.
And good habits start with removing unnecessary friction.

If this post made you rethink even one part of your workflow, then it did its job.

>*Stay curious, automate the boring stuff and happy hacking !*
{: .prompt-danger}