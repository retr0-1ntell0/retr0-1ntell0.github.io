---
title: FlagsFlagsFlags
date: 2025-05-26 22:10:11 -5000
categories: [CTF, Nahamcon 2025, Reverse Engineering]
tags: [reverse engineering]
toc: true
comments: false
image:
  path: Assets/Pictures/CTF/Nahamcon-2025/logo/mod-logo.webp
  lqip: data:image/webp
---

## Description
Author: @Kkevsterrr  
  
Did you see the strings? One of those is right, I can just feel it.

## Reversing

> Disclaimer: 
>
> I am not a professional reverse engineer, I just love it and I love CTFs.
> Not sure if this was the correct way to solve it, as I had to brute force the flag.
{: .prompt-warning }

First we downloaded the `flagsflagsflags` binary. Then we did some static analysis:
```bash
> file flagsflagsflags                  
flagsflagsflags: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, no section header
```
{: .nolineno }
The binary was packed so we could not decompile it properly as shown here we can see the `upx` info and even header at the beginning of running `strings flagsflagsflags`
![packaged](Assets/Pictures/CTF/Nahamcon-2025/packaged.png)

More info to confirm shown below:
![bin=packed](Assets/Pictures/CTF/Nahamcon-2025/packaging-of-bin.png)

We unpacked it first:
```bash
> upx -d flagsflagsflags 
```
{: .nolineno }
Then we can do more enumeration on it, as even the file command displays more data:
```bash
> file flagsflagsflags                                        
flagsflagsflags: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, BuildID[sha1]=8bbcb5450afeba98d27154e01464d3e4888218b7, stripped
```
{: .nolineno }
We then can see more interesting strings and we noticed we could grep the flags:
![flags-in-binary](Assets/Pictures/CTF/Nahamcon-2025/flags-in-binary.png)

We then stored all the strings matching `flag{*}` inside a file called `string-flags.txt`
using the following command to count the number of lines we got in the binary:
```bash
> strings ./unpacked-flagsflagsflags | grep -o 'flag{[^}]*}' | wc -l
100000
```
{: .nolineno }
We needed it stored in a file, so we did the following:
```bash
> strings ./unpacked-flagsflagsflags | grep -o 'flag{[^}]*}' > string-flags.txt  # storing the flags
wc -l string-flags.txt       # counting the lines
```
Here we verify that we got all the flags stored:
![checking](Assets/Pictures/CTF/Nahamcon-2025/verification.png)

This seems like a brute force type of attack, since when even running the binary it ask us for a flag and we have a 100000 flags stored in the binary. But only one of them needs to match in order to retrieve the flag. :
![running-script](Assets/Pictures/CTF/Nahamcon-2025/running-script.png)

So here we made a script called `script.py` to brute force the flag. 
Due to the amount of flags we have stored in the binary, we will need to use threads to speed up the process and subprocess to run the binary and insert the flags one by one.
```python
import threading
import subprocess
from queue import Queue

NUM_THREADS = 100
BINARY_PATH = './flagsflagsflags'
FLAG_FILE = 'string-flags.txt'

# Shared queue of flags
flag_queue = Queue()
found_flag = threading.Event()

def worker(thread_id):
    while not flag_queue.empty() and not found_flag.is_set():
        flag = flag_queue.get().strip()
        print(f"[Thread-{thread_id}] Testing: {flag}")
        try:
            result = subprocess.run(
                [BINARY_PATH],
                input=flag + '\n',
                text=True,
                capture_output=True
            )
            if "Correct" in result.stdout:
                print(f"\n[💀] Flag found by Thread-{thread_id}: {flag}\n")
                found_flag.set()
        except Exception as e:
            print(f"[!] Error testing flag {flag}: {e}")
        finally:
            flag_queue.task_done()

def main():
    # Load flags into the queue
    with open(FLAG_FILE, 'r') as f: 
        for line in f:
            flag_queue.put(line)

    # Create and start threads
    threads = []
    for i in range(NUM_THREADS):
        t = threading.Thread(target=worker, args=(i,))
        t.start()
        threads.append(t)

    # Wait for queue to be empty or a flag to be found
    flag_queue.join()

    # Ensure all threads exit
    for t in threads:
        t.join()

    if not found_flag.is_set():
        print("\n[-] No valid flag found.\n")

if __name__ == "__main__":  
    main()
```
{: .nolineno }

Running our script disclosed the flag as shown below.

Solution:
![flag](Assets/Pictures/CTF/Nahamcon-2025/flag-2.png)

> Flag
> 
> flag{20dec9363618a397e9b2a0bf5a35b2e3}
{: .prompt-info }

Happy hacking ! 😊