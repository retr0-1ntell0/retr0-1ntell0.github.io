---
title: My First CTF 
date: 2025-05-26 22:10:11 -5000
categories: [CTF, Nahamcon 2025, Web]
tags: [web]
toc: true
comments: false
image:
  path: Assets/Pictures/CTF/Nahamcon-2025/logo/mod-logo.webp
  lqip: data:image/webp
---

## Description
Author: BuildHackSecure @ HackingHub  
  
HackingHub has provided this CTF challenge!  
  
On second thoughts I should have probably called this challenge "Nz Gjstu DUG"

## Notes
Here is the home page:
![home](Assets/Pictures/CTF/Nahamcon-2025/home.png)

We decided to fuzz and here is what we found:

![dirs](Assets/Pictures/CTF/Nahamcon-2025/dirs.png)

Now accessing the `flag.txt` we get trolled !
![trolled](Assets/Pictures/CTF/Nahamcon-2025/trolled.png)

Using the hint displayed on the home page, `rotten` so we tried different rot cyphers.
Using the weird sentence in the challenge description, `"Nz Gjstu DUG"` and deciphering it using `ROT-1` gave us the following:
![haha](Assets/Pictures/CTF/Nahamcon-2025/HaHa.png)

So going to the endpoint `gmbh.uyu` which is the `flag.txt` file encrypted using `ROT-1` cypher.

Here is the encrypted `flag.txt` done here[^1]
![rot-1](Assets/Pictures/CTF/Nahamcon-2025/rot-1.png)

So we then visited `http://challenge.nahamcon.com:31441/gmbh.uyu` it downloaded the `flag.txt` file on our machine.

Here is the flag downloaded:

![flag](Assets/Pictures/CTF/Nahamcon-2025/final-flag.png)

> Flag
>
> flag{b67779a5cfca7f1dd120a075a633afe9}
{: .prompt-info }

Happy hacking ! 😊


## References
[^1]: https://www.dcode.fr/rot1-cipher
