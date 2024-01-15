1. `curl 10.13.37.11 -I`, seems like theres no DNS or location
2. `nmap -sVC 10.13.37.11`, mapping the server and it looks like 3 port are open, `22` SSH, `80` HTTP, and `5000` Werkzeug
3. this website runs on wordpress `wpscan --url 10.13.37.11` shows nothing interesting except `XML-RPC` endpoint, visiting the endpoint return an error like `XML-RPC could only accept POST request` so i decided to 
4. this website run on `WordPress version 5.4`

# Flag #1 Plain Sight
1. read the source, the flag is there in the comment
2. flag: `AKERVA{Ikn0w_F0rgoTTEN#CoMmeNts}`
