---
title: Deflation Gangster
date: 2025-05-25 22:10:11 -5000
categories: [CTF, Nahamcon 2025, Reverse Engineering]
tags: [Reverse Engineering]
toc: true
comments: false
---

## Description
Author: @Kkevsterrr  
  
Like American Gangster, but for other stuff.

## Reversing
We downloaded the `gangster.zip` file and unzipped it.
We have access to this folder:
![note](Assets/Pictures/CTF/Nahamcon-2025/Notes-1.png)

Here is more about the content and type of file we are dealing with:
![notes-2](Assets/Pictures/CTF/Nahamcon-2025/Notes-2.png)

Turns out I went too far, so i came back to the zip file to read the metadata, and I noticed this long strings and luckily tried to decrypt it using `base64` and it gave us the flag. This was interesting and more of intuition engineering then reverse engineering 🤣
![disclosed-flag](Assets/Pictures/CTF/Nahamcon-2025/flag-disclosed-2.png)

We solved the challenge and got the flag. This was fast and easy, but I was not expecting this to be so easy.

> Flag
> flag{af1150f07f900872e162e230d0ef8f94}
{: .prompt-info }
