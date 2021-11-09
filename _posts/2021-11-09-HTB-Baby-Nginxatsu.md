---
title: "HTB: Nginxatsu"
header.image: "/assets/images/avatar_black.png"
#last_modified_at: 2021-11-09T14:19:00+01:00
categories:
  - HTB
tags:
  - Easy-Challenge
  - Web
  - OWASP Top 10
  - Broken Authentication
---

# Quick summary
A easy challenge in which we find a misconfigured instance that allows us to grab a backup of the SQLite DB.

## Description
> Can you find a way to login as the administrator of the website and free nginxatsu?

# Enumeration
Visiting the website redirects to `/auth/login`:
[![Visiting website](/assets/htb/baby_nginxatsu/website.png)](/assets/htb/baby_nginxatsu/website.png)

Entering a value that's not a email address:
[![Invalid email](/assets/htb/baby_nginxatsu/invalid_email.png)](/assets/htb/baby_nginxatsu/invalid_email.png)

Entering a random email address gives a error that the account can't be found.

There's a link at the bottom of the page to **Create a new account**.

Creating a new account and logging in.

## Nginx config file
[![Logged in](/assets/htb/baby_nginxatsu/logged_in.png)](/assets/htb/baby_nginxatsu/logged_in.png)
The page looks like a site where the user can generate a nginx config file.<br>

Without changing any of the settings and clicking on **Generate Config** the site generates a new button at the bottom of the page:<br>
[![Generated config](/assets/htb/baby_nginxatsu/generated_config.png)](/assets/htb/baby_nginxatsu/generated_config.png)

Looking at the config the site redirects to `/config/51` where `51` was the number of the generated file.
Content of `/config/51`:

```nginx
nginxatsu

Look at your own nginx config file
Config
Raw Config

user www;
pid /run/nginx.pid;
error_log /dev/stderr info;

events {
    worker_connections 1024;
}

http {
    server_tokens off;

    charset utf-8;
    keepalive_timeout 20s;
    sendfile on;
    tcp_nopush on;
    client_max_body_size 2M;

    include  /etc/nginx/mime.types;

    server {
        listen 80;
        server_name _;

        index index.php;
        root /www/public;

        # We sure hope so that we don't spill any secrets
        # within the open directory on /storage
        
        location /storage {
            autoindex on;
        }
        
        location /www {
            alias /www;
        }
        
        location / {
            try_files $uri $uri/ /index.php?$query_string;
            location ~ \.php$ {
                try_files $uri =404;
                fastcgi_pass unix:/run/php-fpm.sock;
                fastcgi_index index.php;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                include fastcgi_params;
            }
        }
    }
}
```
Interesting comment in the file indicates that `/storage` has some secrets and is a open directory.

## Accessing /storage
Visiting the page `/storage/` shows a Directory Listing of the server.
There's a interesting file named `v1_db_backup_1604123342.tar.gz`.
[![Directory listing](/assets/htb/baby_nginxatsu/backup.png)](/assets/htb/baby_nginxatsu/backup.png)

Downloaded the file using `wget`:
```bash
$ wget http://138.68.131.63:30222/storage/v1_db_backup_1604123342.tar.gz
```

Examining the file to see a sqlite database.
```bash
$ tar -tvf v1_db_backup_1604123342.tar.gz
```

Decompressing the file:
```bash
$ tar -xvf v1_db_backup_1604123342.tar.gz
```

[![DB Backup](/assets/htb/baby_nginxatsu/db_backup.png)](/assets/htb/baby_nginxatsu/db_backup.png)

## Enumerating the database
Since I haven't used SQLite3 before I had to research the tool.
Found the following few commands that will help me in the future:

```
.tables --Shows the tables
PRAGMA table_info(tablename); --Shows the columns of the table
.headers on --When running a SELECT the column names will print
.mode columns --Will pretty up the output of SELECT 
```
[![SQLite3](/assets/htb/baby_nginxatsu/sqlite.png)](/assets/htb/baby_nginxatsu/sqlite.png)

My full SQL enumeration:

```
.tables --To print out the tablenames
.header on --Set a header to pretty up the output
SELECT * FROM users; --To get a full output
SELECT email,password FROM users; --To get the info I need
```

The name of the users indicate that one is the administrator:

|email|password hash|
|---|---|
|nginxatsu-adm-01@makelarid.es|e7816e9a10590b1e33b87ec2fa65e6cd|

### Cracking the hashes
Using [crackstation.net](https://crackstation.net/){:target="_blank"}{:rel="noopener noreferrer"} found the password:<br>
`adminadmin1`

Logging in with the credentials got the flag!<br>
`nginxatsu-adm-01@makelarid.es`:`adminadmin1`<br>

# Pwned confirmation
[![Pwned confirmation](/assets/htb/baby_nginxatsu/pwned_confirmation.png)](/assets/htb/baby_nginxatsu/pwned_confirmation.png)
