---
title: Screenshot
date: 2025-05-25 22:10:11 -5000
categories: [CTF, Nahamcon 2025, WarmUps]
tags: [warmups] 
toc: true
comments: false
---

### Description
Author: @John Hammond

Oh shoot! I accidentally took a screenshot just as I accidentally opened the dump of a `flag.zip` file in a text editor! Whoopsies, what a crazy accidental accident that just accidented!  
  
Well anyway, I think I remember the password was just **`password`**!

### Exploitation
Here is the image `Scrennshot.png` we downloaded:
![accessing-the-chal](Assets/Pictures/CTF/Nahamcon-2025/Screenshot.png)
Here is the extraction of the image:
![accessing-the-chal](Assets/Pictures/CTF/Nahamcon-2025/extracted-text.png)
We downloaded the image and we can see as shown below an hexdump that we formated after extracting it using[^1] and fixed any formatting error:
```text
00000000: 504b 0304 3300 0100 6300 2f02 b55a 0000
00000010: 0000 4300 0000 2700 0000 0800 0b00 666с
00000020: 6167 2e74 7874 0199 0700 0200 4145 0300
00000030: 003d 42ff d1b3 5f95 0314 24f6 8b65 c3f5
00000040: 7669 f14e 8df0 003f e240 b3ac 3364 859e
00000050: 4c2d bc3c 36f2 d4ac c403 7613 85af e4e3
00000060: f90f bd29 d91b 614b a2c6 efde 11b7 1bcc
00000070: 907a 72ed 504b 0102 3f03 3300 0100 6300
00000080: 2f02 b55a 0000 0000 4300 0000 2700 0000
00000090: 0800 2f00 0000 0000 0000 2080 b481 0000
000000a0: 0000 666с 6167 2e74 7874 0a00 2000 0000
000000b0: 0000 0100 1800 8213 8543 07ca db01 0000
000000c0: 0000 0000 0000 0000 0000 0000 0000 0199
000000d0: 0700 0200 4145 0300 0050 4b05 0600 0000
000000e0: 0001 0001 0065 0000 0074 0000 0000 00
```
We extracted it using[^1] and fixed any error:
Then we made the following python script to solve it:
```python

hex_data = """
504b 0304 3300 0100 6300 2f02 b55a 0000
0000 4300 0000 2700 0000 0800 0b00 666c
6167 2e74 7874 0199 0700 0200 4145 0300
003d 42ff d1b3 5f95 0314 24f6 8b65 c3f5
7669 f14e 8df0 003f e240 b3ac 3364 859e
4c2d bc3c 36f2 d4ac c403 7613 85af e4e3
f90f bd29 d91b 614b a2c6 efde 11b7 1bcc
907a 72ed 504b 0102 3f03 3300 0100 6300
2f02 b55a 0000 0000 4300 0000 2700 0000
0800 2f00 0000 0000 0000 2080 b481 0000
0000 666c 6167 2e74 7874 0a00 2000 0000
0000 0100 1800 8213 8543 07ca db01 0000
0000 0000 0000 0000 0000 0000 0000 0199
0700 0200 4145 0300 0050 4b05 0600 0000
0000 0001 0001 0065 0000 0074 0000 0000
00
"""

hex_data = hex_data.replace("\n", "").replace(" ", "")

binary_data = bytes.fromhex(hex_data)

with open("recovered.zip", "wb") as zip_file:
    zip_file.write(binary_data)

print("ZIP file has been recovered as 'recovered.zip'.")

```
Once solved, we got our zip:
```bash
> file recovered.zip 
recovered.zip: Zip archive data, at least v5.1 to extract, compression method=AES Encrypted
```

One could also use `cyberchef`[^2]
![accessing-the-chal](Assets/Pictures/CTF/Nahamcon-2025/cyber-chef.png)
On the console to unzip we encountered some errors:
```bash
> unzip recovered.zip 
Archive:  recovered.zip
error [recovered.zip]:  missing 14221096 bytes in zipfile
  (attempting to process anyway)
error [recovered.zip]:  attempt to seek before beginning of zipfile
  (please check that you have transferred or created the zipfile in the
  appropriate BINARY mode and that you have compiled UnZip properly)
```

To solve it we ran the following command:
```bash
zip -FF recovered.zip --out fixed.zip
```
This attempts to "fix" a damaged ZIP archive by reassembling any recoverable parts. Then once we have our fixed zip, we could unzip it.
Using `unzip` we still got yet another error due to version compatibility:
```bash
> unzip fixed.zip 
Archive:  fixed.zip
   skipping: flag.txt                need PK compat. v5.1 (can do v4.6)
```
For this we changed tools to use `7z`
{% raw %}
```bash
> 7z x fixed.zip 

7-Zip 24.09 (x64) : Copyright (c) 1999-2024 Igor Pavlov : 2024-11-29
 64-bit locale=en_US.UTF-8 Threads:20 OPEN_MAX:1024, ASM

Scanning the drive for archives:
1 file, 239 bytes (1 KiB)

Extracting archive: fixed.zip
--
Path = fixed.zip
Type = zip
Physical Size = 239

    
Enter password:[INFO] [DkImageLoader] 1 containers created in 0 ms
[INFO] [DkImageLoader] after sorting:  0 ms
password

Everything is Ok

Size:       39
Compressed: 239
[INFO] [DkImageLoader] 1 containers created in 0 ms                                                          
[INFO] [DkImageLoader] after sorting:  0 ms
```
{% endraw %}

We were able to recover the file and unzip the file successfully and read flag.txt file that contained the flag.

> Flag
>
>flag{907e5bb257cd5fc818e88a13622f3d46}
{: .prompt-info }   



## References

[^1]: https://www.imagetotext.info/

[^2]: https://gchq.github.io/CyberChef/#recipe=From_Hexdump()&input=MDAwMDAwMDA6IDUwNGIgMDMwNCAzMzAwIDAxMDAgNjMwMCAyZjAyIGI1NWEgMDAwMAowMDAwMDAxMDogMDAwMCA0MzAwIDAwMDAgMjcwMCAwMDAwIDA4MDAgMGIwMCA2NjbRgQowMDAwMDAyMDogNjE2NyAyZTc0IDc4NzQgMDE5OSAwNzAwIDAyMDAgNDE0NSAwMzAwCjAwMDAwMDMwOiAwMDNkIDQyZmYgZDFiMyA1Zjk1IDAzMTQgMjRmNiA4YjY1IGMzZjUKMDAwMDAwNDA6IDc2NjkgZjE0ZSA4ZGYwIDAwM2YgZTI0MCBiM2FjIDMzNjQgODU5ZQowMDAwMDA1MDogNGMyZCBiYzNjIDM2ZjIgZDRhYyBjNDAzIDc2MTMgODVhZiBlNGUzCjAwMDAwMDYwOiBmOTBmIGJkMjkgZDkxYiA2MTRiIGEyYzYgZWZkZSAxMWI3IDFiY2MKMDAwMDAwNzA6IDkwN2EgNzJlZCA1MDRiIDAxMDIgM2YwMyAzMzAwIDAxMDAgNjMwMAowMDAwMDA4MDogMmYwMiBiNTVhIDAwMDAgMDAwMCA0MzAwIDAwMDAgMjcwMCAwMDAwCjAwMDAwMDkwOiAwODAwIDJmMDAgMDAwMCAwMDAwIDAwMDAgMjA4MCBiNDgxIDAwMDAKMDAwMDAwYTA6IDAwMDAgNjY20YEgNjE2NyAyZTc0IDc4NzQgMGEwMCAyMDAwIDAwMDAKMDAwMDAwYjA6IDAwMDAgMDEwMCAxODAwIDgyMTMgODU0MyAwN2NhIGRiMDEgMDAwMAowMDAwMDBjMDogMDAwMCAwMDAwIDAwMDAgMDAwMCAwMDAwIDAwMDAgMDAwMCAwMTk5CjAwMDAwMGQwOiAwNzAwIDAyMDAgNDE0NSAwMzAwIDAwNTAgNGIwNSAwNjAwIDAwMDAKMDAwMDAwZTA6IDAwMDEgMDAwMSAwMDY1IDAwMDAgMDA3NCAwMDAwIDAwMDAgMDAK&oeol=NEL
