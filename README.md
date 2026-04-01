# VA-W3-PwnTillDawn-Portal-Writeup

## PwnTillDawn Target Profile
- **Machine/Lab Name**: Portal
- **Target IP**: 10.150.150.12
- **Difficulty**: Easy
- **Operating System**: Linux

## 1. Reconnaissance
- To begin, a scan was performed on the PwnTillDawn network where the Target IP ```10.150.150.12``` was found and selected with the nmap command ```nmap -sn 10.150.150.10-254```
- This target IP was then entered into the active targets section in the PwnTillDawn website where its IP address and operating system (Linux) was identified.
## 2. Scanning
- A detailed an aggressive scan was performed on the target IP with ```nmap -sC -sV -p- 10.150.150.12```
  - This command simply scans every possible port on the target, identifies what services are running, and runs basic vulnerability/info scripts on those services
Scan Results:
- Scan completed in 436.36 seconds.
- Ports 21 and 22 open
  - Port 21 | STATE: OPEN | SERVICE: FTP | VERSION: vsftpd 2.0.8 or later
  - Port 22 | STATE: OPEN | SERVICE: SSH | VERSION: OpenSSH 8.2p1 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
## 3. Gaining Access
- Conveniently, the ```ftp-syst``` output revealed the version ```vsftpd 2.3.4``` which specifically has a vulnerability that can be exploited 
  - The exploit was that a server would trigger a listener on port 6200 if a smiley face ```:)``` was used at the end of a username when logging into the ftp session
- Therefore an FTP session was established with ```ftp 10.150.150.12``` where the output produced a banner that said:  ```"Through the portal... - into nothingness or bliss?"``` (not important but just a neat little notice) along with a prompt to login with a username and password
 - Using the exploit I entered ```user:)``` as the username and ```pass``` as the password (password doesn't matter in this case)
- Finally a netcat connection was established to the target with the command ```nc -nv 10.150.150.12 6200``` on the port ```6200```
  - This resulted in a "blind" interactive shell
  - Running ```id``` and ```whoami``` confirmed that the shell was running with root privileges immediately upon connection
    - ```id``` command result: ```uid=0(root) gid=0(root) groups=0(root)```
    - ```whoami``` command result: ```root```
## 4. Escalate Privileges
- In this case Privilege Escalation was not required as the vulnerability in the vsftpd 2.3.4 service produced a shell with root permissions.
- Running ```ls``` on the root directory produced two output items: ```FLAG1.txt``` and ```snap```
  - ```cat``` was then ran for the file contents of ```FLAG1.txt``` to obtain the singular flag needed for this PwnTillDawn target
  - FLAG1.txt: ```5ee499eb5d0b8e4269b13483e57adaa0b3815f48```
## 5. Maintain Access
- To avoid being in the unstable backdoor port 6200 I ensured persistent access by chainging the root password to login in through the standard SSH service **(Port 22)**
  - ```passwd root``` was entered that allowed me to enter a new password twice (``p@ssword``)
  - Then a SSH connection was established with ```ssh root@10.150.150.12```
## 6. Clearing Tracks
- Logs were cleared with ```history -c``` to remove traces of exploitation commands
- The FTP connection used to trigger the backdoor was closed with ```exit``` along with the temrination of the 6200 port listener after stable SSH access was confirmed

