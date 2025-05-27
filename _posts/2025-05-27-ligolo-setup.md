---
title: Ligolo Setup
date: 2025-05-27 13:13:13 -5000
categories: [Tools, Tunneling]
tags: [penetration testing] 
toc: true
comments: false
pin: true
description: Ligolo Setup is a simple tool that automates the process of setting up the interfaces and routes to speed up your workflow.
---


    $$\      $$\                  $$\                $$$$$$\           $$\    $$\   $$\          
    $$ |     \__|                 $$ |              $$  __$$\          $$ |   $$ |  $$ |         
    $$ |     $$\ $$$$$$\  $$$$$$\ $$ |$$$$$$\       $$ /  \__|$$$$$$\$$$$$$\  $$ |  $$ |$$$$$$\  
    $$ |     $$ $$  __$$\$$  __$$\$$ $$  __$$\$$$$$$\$$$$$$\ $$  __$$\_$$  _| $$ |  $$ $$  __$$\ 
    $$ |     $$ $$ /  $$ $$ /  $$ $$ $$ /  $$ \______\____$$\$$$$$$$$ |$$ |   $$ |  $$ $$ /  $$ |
    $$ |     $$ $$ |  $$ $$ |  $$ $$ $$ |  $$ |     $$\   $$ $$   ____|$$ |$$\$$ |  $$ $$ |  $$ |
    $$$$$$$$\$$ \$$$$$$$ \$$$$$$  $$ \$$$$$$  |     \$$$$$$  \$$$$$$$\ \$$$$  \$$$$$$  $$$$$$$  |
    \________\__|\____$$ |\______/\__|\______/       \______/ \_______| \____/ \______/$$  ____/ 
                $$\   $$ |                                                             $$ |      
                \$$$$$$  |                                                             $$ |      
                 \______/                                                              \__|     



## Description

Ligolo is a tool that helps when we need to pivot through a network. I really love this tool as it makes a lot of things easier. But the part I dislike the most is that the set up is not automated. Which is understandable due to all the work that needs to be done to set up the tool. That's why to help myself skip the trouble of setting up the interface and adding the route every time I need to use it I decided to write a tool, I called `ligolo-setup`[^1] that will help me automate the process of setting up the interfaces and routes to speed up my workflow.  

## Disclaimer

This is only for educational purposes. I am not responsible for any damage that may occur to your network or computer. Use this tool at your own risk even if there is no damage. Make sure to read the documentation of `ligolo-ng`[^2] and this documentation before using those tools. You can find all the links in the [references](#references) section.
  
## Requirement

The only requirement so far is to have root access or password to be able to make it work. If you are having issues running the commands you are probably missing `iproute` or `iproute2`, `awk`, `grep`, `xargs` and `nl` packages depending on your OS. Of course, you need to have `ligolo-ng` installed and renaming the name of the binary to `ligolo-proxy` in your executable environment variable `PATH`.

On `Debian/Ubuntu` based distributions:
```
sudo apt update
sudo apt install iproute2
```
{: .nolineno}

On `Red Hat/CentOS/Fedora-based` distributions:
```
sudo yum install iproute
```
{: .nolineno}

or (on newer versions of `Fedora`):
```
sudo dnf install iproute
```
{: .nolineno}

On `Arch Linux/Endeavor OS/Manjaro`:
```
sudo pacman -S iproute2
```
{: .nolineno}
or
```
yay -S iproute2
```
{: .nolineno}

## Usage

Very easy to use, so that you don't have to type these everytime. You can add one or delete one. Your choice. Once installed, give it executable permissions. Also make sure to have `ligolo-ng` installed and the proxy having the name `ligolo-proxy` in your PATH.

The `ligolo-ng` binary releases can be found here [^3] 
```shell
chmod +x ligolo-setup.sh
```
{: .nolineno}

Once this step is done we are ready to roll, now just execute it and make a choice:
```shell
./ligolo-setup.sh
```
{: .nolineno}

Once launched, we are presented with the following screen with options showed in order of execution:

![homepage](Assets/Pictures/ligolo-setup/home-page.png){:width="500px"}

From let's walk through all the different options:
- `[1]` : This options automates the process of adding a new `tuntap` device before using `ligolo-ng`. This created `tuntap` device is the device that would be used by `ligolo-ng` for tunneling. This reduces the stress of typing that command every time we need to add a new device.
- `[2]` : This is self explanatory, whenever we are done with the `tuntap` interface, we can delete it. In case of 2 devices we would be presented with a choice of the interface to delete. At the time of the writing this script only supports adding and removing 2 interfaces.
- `[3]` : Here is where we will add the route we want to pivot to usually in **CIDR(Classless Inter-Domain Routing**) format. eg: `10.10.10.0/24`. In case of a double pivot we can always come back here and add a new route using this option.
- `[4]` : Here we are face with the option to delete the routes we previously added. Just like the interfaces, when we got multiple routes, the script ask you which one to delete.
- `[5]` : Finally we can start out proxy to start interacting with `ligolo-ng` binary. In this step, there is a check that will not let you use this option without setting a `tuntap` interface using the option `[1]`

Here are little helpers and what they do:
- `[i/I]` : This options will help you visualize the interface in the `ifconfig` format to see if there is any `tuntap` device set or to see the name of the device that the tool name by default `ligolo`.
- `[c/C]` : Whenever the screen becomes messy, we can use this just to clear the screen, nothing fancy. A regular `CTRL+L` or `clear` command.
- `[q/Q]` : Best way to quit this tool as it does the clean up process of the certificate directories added by `ligolo-ng`.




## References

[^1]: https://github.com/retr0-1ntell0/ligolo_setup

[^2]: https://docs.ligolo.ng/

[^3]: https://github.com/nicocha30/ligolo-ng/releases


