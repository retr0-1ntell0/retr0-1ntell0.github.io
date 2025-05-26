---
title: Puzzle Pieces
date: 2025-05-25 22:10:11 -5000
categories: [CTF, Nahamcon 2025, OSINT]
tags: [OSINT]
toc: true
comments: false
---

## Description
Author: Nordgaren  
  
Well, I accidentally put the important data into a bunch of executables.  
  
It was fine, until my cat stepped on my keyboard and renamed them all!  
  
Can you help me recover the important data?

**NOTE, the password for the archive is `nahamcon-2025-ctf`**

## Exploitation

After unzipping the downloaded file with the given password of `nahamcon-2025-ctf`:
```bash
7z e ctf_challenge_files.7z
```
We started gathering the metadata of the file. Turns out they are all windows PE32 executable as shown below:
![metadata](Assets/Pictures/CTF/Nahamcon-2025/files-metadata.png)
Then we look deeper into the metadata using `exiftool`
We discovered the following line containing the order in which we compile the files in order to get the flags:
```bash
PDB File Name                   : xxxxxxxxxxx.pdb
```
So we did run the following command to filter the `PDB File Name` and the name of the file in order to get the order of execution:
```bash
> ls -l *.exe && exiftool * | grep 'PDB File Name'
```
Here is what the ouput looks like:
![execution-order](Assets/Pictures/CTF/Nahamcon-2025/filtering.png)

Having that order, we compiled the files one at the time in order to get the flag parts by parts.
Here is the screenshot of the execution order:
![execution-order](Assets/Pictures/CTF/Nahamcon-2025/flag-disclosed.png)

This concatenated the flag parts and we got the flag:

> Flag
>
> flag{512faff5e7d89c9b8bd4b9517af9bfaa}
{: .prompt-info }

