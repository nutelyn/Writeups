images here
# Reconnaissance
1. `curl -I 10.10.11.245` to find a possible DNS name, and i found `surveillance.htb` add it to `/etc/hosts`
2. scanning discover a `/admin` which lead to `/admin/login`
3. system is running `craft CMS` version unknown, but there's a recent discovery of RCE, details can be found here `https://gist.github.com/to016/b796ca3275fa11b5ab9594b1522f7226`

# User Escalation
1. based on the POC, `python3 craftcms-cve.py http://surveillance.htb` to send the payload
3. and i got a reverse connection, after exploring quite a while i found