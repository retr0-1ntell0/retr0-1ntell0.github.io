---
title: Sending Mixed Signals
date: 2025-05-25 22:10:11 -5000
categories: [CTF, Nahamcon 2025, OSINT]
tags: [osint]
toc: true
comments: false
---

## Description
Author: @Jstith  
  
Turns out the Houthi PC Small Group was only the tip of the iceberg...

**REMINDER: Brute force guessing is NOT permitted.**

**Press the Start button begin this challenge.**

## OSINT Time
Here is what we got after strating and accessing the webpage:
![home-mixed-signals](Assets/Pictures/CTF/Nahamcon-2025/home-mixed-signals.png)

### Part 1

> Find the hard-coded credential in the application used to encrypt log files. (format `jUStHEdATA`, no quotes)
{: .prompt-question }

So we did some research online looking for `Find the hard-coded credential in the application used to encrypt log files signal app scandal` 
We found this link[^1] for the first answer which was the following key as shown in the picture, we found the repository and went there for more enumeration[^2]
![log-file](Assets/Pictures/CTF/Nahamcon-2025/lofigle-key.png)
Here is the hard-coded credential in the application used to encrypt log files:
```text
enRR8UVVywXYbFkqU#QDPRkO
```

### Part 2

> Find the email address of the developer who added the hard-coded credential from question one to the code base (format `name@email.site`)
{: .prompt-question }

Now we can go further and look for the email account of the developer wjp added the hard-coded credentials from question one to the code base. For that we kept reading the article and found the following line:
![disclosure-emails.png](Assets/Pictures/CTF/Nahamcon-2025/disclosure-emails.png)
Then we had to clone the repo in order to see who did what:
```bash
git clone https://github.com/micahflee/TM-SGNL-Android
```
So we needed the commit hash in order to disclose that info, we got it the website here[^3] as shown below:
![commit-hash.png](Assets/Pictures/CTF/Nahamcon-2025/commit-hash.png)
Then we can run the following command, since we have a huge repository we need to specify the string in which we want to search for:
```bash
git log -S "enRR8UVVywXYbFkqU#QDPRkO"
```
Here is what we got:
![commit-hash](Assets/Pictures/CTF/Nahamcon-2025/mail-discovered.png)

We found the correct email:
```text
moti@telemessage.com
```

### Part 3

> Find the first published version of the application that contained the hard-coded credential from question one (case sensitive, format Word_#.#.#......).
{: .prompt-question }

We ran:
```bash
git log -S "enRR8UVVywXYbFkqU#QDPRkO" --oneline
```
And got:
![release](Assets/Pictures/CTF/Nahamcon-2025/release.png)
Adding this we got the last answer correct:
![flag](Assets/Pictures/CTF/Nahamcon-2025/flag-mixed.png)
Got the flag and solved the challenge.

> Flag
>
> flag{96143e18131e48f4c937719992b742d7}
{: .prompt-info }


## References
[^1]: https://micahflee.com/heres-the-source-code-for-the-unofficial-signal-app-used-by-trump-officials/

[^2]: https://github.com/micahflee/TM-SGNL-Android/commit/05665a465a21c61529b76868edb7032c3ed81374

[^3]: https://github.com/micahflee/TM-SGNL-Android/commit/05665a465a21c61529b76868edb7032c3ed81374
