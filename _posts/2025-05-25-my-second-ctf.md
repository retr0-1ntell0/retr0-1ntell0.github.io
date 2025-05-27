---
title: My Second CTF 
date: 2025-05-26 22:10:11 -5000
categories: [CTF, Nahamcon 2025, Web]
tags: [web]
toc: true
comments: false
---

## Description
Author: BuildHackSecure @ HackingHub  
  
HackingHub has provided this CTF challenge!  
  
**NOTE, this challenge requires some content discovery but only use the `wordlist.txt` file we've supplied to avoid wasting your time!**  
  
Special thanks to [HackingHub](https://hackinghub.io/) for the sponsorship and support of the NahamCon CTF!

## Exploitation
Here is the home page:
![home](Assets/Pictures/CTF/Nahamcon-2025/home.png)
We downloaded the `wordlist.txt` file for the challenge.
We used it and made a rot 2 with the wordlist.
We kept the same logic of looking at the hints shown to us with the picture of the webpage.
Since the the picture is saying `One step more rotten` now we got the following wordlist `rot-2` encrypted, being one more step from the previous one.

Here is the script, that encrypted all the words in the wordlist into a `ROT-2` cypher.

```python
def apply_rot2_to_string(text):
    rotated_text = []
    for char in text:
        if 'a' <= char <= 'z':
            rotated_text.append(chr(((ord(char) - ord('a') + 2) % 26) + ord('a')))
        elif 'A' <= char <= 'Z':
            rotated_text.append(chr(((ord(char) - ord('A') + 2) % 26) + ord('A')))
        else:
            rotated_text.append(char)
    return ''.join(rotated_text)


def convert_file_to_rot2(input_file, output_file):
    try:
        with open(input_file, 'r') as infile, open(output_file, 'w') as outfile:
            for line in infile:
                rotated_line = apply_rot2_to_string(line.strip())  
                outfile.write(rotated_line + '\n')
        print(f"Conversion successful! Check the output file: {output_file}")
    except FileNotFoundError:
        print(f"The file {input_file} does not exist.")
    except Exception as e:
        print(f"An error occurred: {e}")

# Usage Example
input_filename = 'wordlist.txt' 
output_filename = 'rot-2-output.txt' 

convert_file_to_rot2(input_filename, output_filename)
```
Running `dirsearch` we found:
![dirs](Assets/Pictures/CTF/Nahamcon-2025/dir-found.png)

Trying to access it, we get the following error:
![[missing-param.png)
So went to burp suite and fire it up !
So having the endpoint of `debug` => `fgdwi` and missing the parameter, send us to use intruder and do the following query to brute force the parameter using the same `ROT-2` encrypted file as the loaded file.
Here is the result of the parameter fuzz:
![finding](Assets/Pictures/CTF/Nahamcon-2025/finding.png)

We added the file `flag.txt` intuitively but turns out it was the right approach, we disclosed the flag by looking at the content Length tab, and the `eqphkto` => `confirm` was the right parameter and we also solve the challenge and grabbed the flag as shown in the response:
![flag](Assets/Pictures/CTF/Nahamcon-2025/flag-3.png)

WE got the flag and solved the challenge.


> Flag
>
> flag{9078bae810c524673a331aeb58fb0ebc}
{: .prompt-info }

Hppy hacking ! 😊


