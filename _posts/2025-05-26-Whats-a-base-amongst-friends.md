---
title: What's a Base Amongst Friends 
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
  
What's a base amongst friends though, really?

### Reversing
Now we downloaded the binary named `whats-a-base`.

First let's analyse the file and look for strings:
```bash
> file whats-a-base 
whats-a-base: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=484060c717cdff4f3fd09b358136996ec7b7feaa, for GNU/Linux 3.2.0, stripped
```
{: .nolineno }
Here are some of the strings in the binary:
```bash
> strings whats-a-base | head -n 50
/lib64/ld-linux-x86-64.so.2
_ITM_deregisterTMCloneTable
__gmon_start__
_ITM_registerTMCloneTable
_Unwind_GetLanguageSpecificData
_Unwind_GetIPInfo
_Unwind_GetRegionStart
_Unwind_SetIP
_Unwind_DeleteException
_Unwind_GetIP
_Unwind_Resume
_Unwind_Backtrace
_Unwind_GetTextRelBase
_Unwind_GetDataRelBase
_Unwind_RaiseException
_Unwind_SetGR
ceilf
pthread_key_delete
sysconf
free
lseek64
statx
pthread_self
sigaction
fcntl
realpath
open64
munmap
memmove
mmap64
__cxa_thread_atexit_impl
poll
__xpg_strerror_r
strlen
read
pthread_attr_getguardsize
readlink
realloc
pthread_attr_destroy
dl_iterate_phdr
getauxval
malloc
__libc_start_main
```
{: .nolineno }

Now let's execute and see what we need to do :
![inputs](Assets/Pictures/CTF/Nahamcon-2025/inputs.png)

So we will need a password in order to get the flag.
The decompiled and graphical debugging tool were a mess:

So we went back to static analysis, with strings that we stored in a file, and looked for the `Invalid Password!` entry here is what we found:
![strings](Assets/Pictures/CTF/Nahamcon-2025/strings.png)
We took that first string:
```text
src/main.rsm7xzr7muqtxsr3m8pfzf6h5ep738ez5ncftss7d1cftskz49qj4zg7n9cizgez5upbzzr7n9cjosg45wqjosg3mu
```
{: .nolineno }
Extracted the following string, right after the `src/main.rs` string, we now have the following string:
```text
m7xzr7muqtxsr3m8pfzf6h5ep738ez5ncftss7d1cftskz49qj4zg7n9cizgez5upbzzr7n9cjosg45wqjosg3mu
```
{: .nolineno }

Trying it on the binary still gave us the same error. But it shows at least that it's part of the code since we found it in the strings and should be input as the password but just not the exact string.
Then we started looking for ways to decrypt it.
So went back to the strings and found the following string:
![z-base-32](Assets/Pictures/CTF/Nahamcon-2025/zbase.png)
We find out that we could use the following website[^1] to decrypt using the `z-base-32` format, and doing so gave us the following result:
![decoding](Assets/Pictures/CTF/Nahamcon-2025/decoding.png)
In plain text it was:
```text
__rust_begin_short_backtrace__rust_end_short_backtraces
```
{: .nolineno }
Running the following against the binary we got the flag and solved the challenge:
![flag](Assets/Pictures/CTF/Nahamcon-2025/flag-revs.png)

> Flag
>
> flag{50768fcb270edc499750ea64dc45ee92}
{: .prompt-info }

## References
[^1]: https://cryptii.com/pipes/z-base-32
