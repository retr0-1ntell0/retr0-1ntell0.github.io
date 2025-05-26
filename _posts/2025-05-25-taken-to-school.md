---
title: Taken to School
date: 2025-05-25 22:10:11 -5000
categories: [CTF, Nahamcon 2025, OSINT]
tags: [osint]
toc: true
comments: false
---

## Description
"I was reading the news this week, and I saw that a student tried to hack a school's computer system!" a worried professor remarked to an IT employee during lunch. "I'm glad we've got people like you keeping our network safe." While Bob the IT admin appreciated the warm comment, his stomach dropped. "Dang it.. I haven't checked that firewall since we set it up months ago!".  
  
IT has pulled a log file of potentially anomalous events detected by the new (albeit poorly tuned) network security software for your school. Based on open-sourced intelligence (OSINT), identify the anomalous entry in the file.  
  
Each log entry contains a single line, including an MD5 hash titled `eventHash`.  
  
The challenge flag is `flag{MD5HASH}` containing the `eventHash` of the anomalous entry.

## Exploitation
We downloaded a file named `network-log.cef`.
First thing we do when we get a file with a weird extension, we enumerate the format using `file`:
```bash
> file network-log.cef
network-log.cef: ASCII text
```
So we can read the file just as we would read any text.
We looked to see how many lines we got, and the magic `500` lines appeared again:
```bash
> wc -l network-log.cef 
500 network-log.cef
```
Clearly we will need a script in order to go through the file and get the needed `eventHash`.
We tried retrieving the uniq `eventHash`, but we got too many candidates as all the hash weere unique duhh :
```bash
grep -o 'eventHash=[a-f0-9]*' network-log.cef | awk -F'=' '{print $2}' | sort | uniq > uniq-eventHash
```

Let's actually look into the data and understand to see maybe we can filter something and look for anomalies.
Here is the filter we ran after looking at the data for a good while.
So we are looking for any action that is allowed and filtering the `threatType` to see which ones comes out.
We ran:
```bash
grep 'threatType' network-log.cef | grep 'act=allowed'
```
Here is the result:
![threat-type](Assets/Pictures/CTF/Nahamcon-2025/threat-type.png)

We still had a lot of content, with different threat type. Then later on we added another filter to get the trojan types:
```bash
grep 'threatType' network-log.cef | grep 'act=allowed' | grep 'trojan'
```
We got two results, thankfully we were not limited to with trials, so we got:
![possible-flags](Assets/Pictures/CTF/Nahamcon-2025/possible-flags.png)
With these two we tried our luck and luckily we got it on the second attempt, the solution was the following:
```bash
eventHash=5b16c7044a22ed3845a0ff408da8afa9
fileName=chemistry_notes.pdf
```

So the result was:
> Flag
>
> flag{5b16c7044a22ed3845a0ff408da8afa9}
{: .prompt-info }
