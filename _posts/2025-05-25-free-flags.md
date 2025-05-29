---
title: Free Flags
date: 2025-05-25 22:10:11 -5000
categories: [CTF, Nahamcon 2025, WarmUps]
tags: [warmups]
toc: true
comments: false
image:
  path: Assets/Pictures/CTF/Nahamcon-2025/logo/mod-logo.webp
  lqip: data:image/webp
---

## Description
Author: @John Hammond

WOW!! Look at all these free flags!!

But... wait a second... only one of them is right??

NOTE, bruteforcing flag submissions is still not permitted. I will put a "max attempts" limit on this challenge at 1:00 PM Pacific to stop participants from automating submissions. There is only one correct flag, you can find a needle in a haystack if you really know what you are looking for.

## Exploitation
So we downloaded the `free_flags.txt` file and it's a huge file of flags as it contains `500` lines with a size of `117 KB`:
```bash
> wc -l free_flags.txt 
500 free_flags.txt
```
{: .nolineno }

From here we went back to the following page[^1] to look at the flag format as shown below:
![accessing-the-chal](Assets/Pictures/CTF/Nahamcon-2025/rules.png)
So we got the following format of flag => `flag\{[0-9a-f]{32}\}`
This is how the following python script that will actually disclose the only flag having the correct format according the regular expression mentioned above, was made:

{% raw %}
```python
import re

def find_valid_flags(file_path):
    flag_pattern = r"flag\{([0-9a-f]{32})\}"

    with open(file_path, 'r') as file:
        content = file.read()

    flags = re.findall(flag_pattern, content)

    return flags

file_path = 'free_flags.txt'

valid_flags = find_valid_flags(file_path)

if valid_flags:
    print("Valid flags found:")
    for flag in valid_flags:
        print(f"flag{{{flag}}}")  
else:
    print("No valid flags found.")
```
{%endraw%}

Running it, disclosed the flag:
```bash
> python script.py 
Valid flags found:
flag{REDACTED}
```
{: .nolineno }


> Flag
>
> flag{ae6b6fb0686ec594652afe9eb6088167}
{: .prompt-info }


## References
[^1]: https://ctf.nahamcon.com/rules
