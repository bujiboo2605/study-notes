## What is FTP
ftp is a port that transfer files between client and sever over the network. Pretty much its a protocol to replace usb driver, a company have employee that wants to upload reports , admins to administrate them , managers who needs to download them and so on , ftp allow the employees to do all the above over the network.We are the ftp client , server is the ftp server , we connect to the server with credentials and upload and download files.

Example ftp client and server service that accept ftp client

| client                          | services                 |
| ------------------------------- | ------------------------ |
| kali linux ftp client           | vsftpd                   |
| filezilla                       | proftpd                  |
| windows command line ftp client | pure-ftpd                |
| linux command line ftp client   | Microsoft ISS FTP server |

### How ftp connection works

![[Pasted image 20261123024454.png]]
 so ftp connection is made with a 3 way hand shake , pretty common
 1. the client send a SYN packet to the server or ftp service 
 2. the server will send a SYN-ACK which is basically server saying i acknowledge your synconize package.
 3. then client send a AC K packet again to establish the connection now there is a tcp connection between client and server
 4. now server will ask for authentication , username and password before logging in.

#### what is happening inside the system?
while on top its just a 3 way ahndshave ftp does more than that at an application level, transport level and network level.

In application level 
- fpt client generate commands
- ftp server interprets those commands
- the server check authentication and permissions
- server accesses the requested filesystem resources
- server send or recieve the file data

In transport level 
- TCP will make a reliable connection
- TCP will handle sequencing, acknowledgement and retransmissions
- FTP uses seperate control and data connection

At network level 
- IP address identify the communication systems 
- Routes the formard packets 
- Firewalls maybe allow or block connection

So the above list shows that ftp doesnt operate alone and do everything alone,TCP is prviding the foundation for ftp to build and work on for reliable transport.

ftp all in 1 guide
https://www.rfc-editor.org/rfc/rfc959.html?utm_source=chatgpt.com

### what a pentester must look for
1. is ftp service accessible
2. Which FTP implementation is running
3. What version is installed
4. Does it allow anonymous authentication
5. Does it expose sensitive directories
6. Am i or users are capable of uploading files
7. is the authentication encrypted
8. Does the service running contain known vulnerabilities or CVEs

Tip - ftp is an non encrypted file transfer which means if a user is connecting thru an unsafe network then the credentials and any data transfer upload and download can be read by a wireshark by analyzing the network packets.
## Control connection - TCP 21
 so ftp uses 2 connection ports port 20 and 21. the 2 ports are divided for 2 different functions 
 1. control connection - port 21 is used for control connection , it is used to carry ftp commands and responses from the server and client. The recieve the command thru port 21 , it understand the commands and proceeds to do what is asked.
 2. data connection - port 20 is used to transfer data , it can be download ,directory listing or upload but all data transfer will go thru port 20 .

### why seperate connection
so there are multiple reasons for the different port and different use. Firstly this allows ftp to maintain command session while transfering files, it provide a very clear distinction between commands that control the session and data that is associated with file operation.

secondly , the behavior of control connection and data connection is different, the control connection remain open until the ftp session is closed. While data connection is closed after individiual data is finished transfering. 

finally , both network connection uses different network endpoints, while control connection used port 21 the data connection endpoint is determined by the ftp transfer mode. Hence why ftp behave differently from many other applicaitons

## WHY IS PORT 20 NOT ALWAYS OPEN

Now this is something i faced , i can see that the port 21 is open but 20 is filtered or closed. Port 20 is usually associated with source port of active mode data connection (what does that mean) -  it means the client will open a command channel or control connection to tell the server which client port to connect back to for transferring data. While on passive mode the server initiates the data connection. What the heck does that mean - it means while port 21 is open it doesnt neccesaryly need port 20 to be open to prove successful ftp service running, if the data connection is passive then it can be programmed to any port such  as port 50001 . 

in a nutshell -
- TCP 21 is ftp control command port
- TCP 20 is active mode server data source port / client must get control connection and designated the server to make connection back to specific port in client end for data transfer
- passive FTP - can be configured to any unused ports
- firewall config can and will affect data transfer
- dynamic ports may appear during passive mode operations

Example scenario - 
when u discover a port 21 and u log in anonymously it return as a success , YAY WE CAN LIST DIRECTORIES ... except the directory listing fail. While the logged in used may not have the power it can also mean that the command connectionworks but data connection can be configured to a passive port or a firewall is blocking the connection.

## Active vs Passive ftp 
Now lets see how active and passive connection are differnt. To boil down active and passive its mainly about who initiates the connection , but why is this even important well -
The importance is it effects  - 
- firewall configs
- NAT behavior
- Network connectivity
- FTP server configuration
- Data transfer troubleshooting

### Active ftp
in active connection the client initiates the control connection (port 21) and the server will initiate the data connection(port 20). The client will call up server and tell yo connect to 10.10.10.50 at port 50000 using ftp command.

==PORT 10,10,10,20,195,80== - ftp port command

the PORT command represents the clients ip address and data port in six comma seperated numbers the last 2 represent the port ( i dont understand this either ).  lets look at the calculation 

port = (p1 x 256) + p2  - which in this case will be (195 x 256) + 80 = 50000
 so the server connects to port 50000

###### Why calculation are needed?? (side quest)
So why are we like sending it with calculation and not just 50000?? makes no sense right well actually it does the port command can only send values from 0 - 255 since 50000 is more than that so to send it we use a simple maths , now what if i wanna connect to a different port instead like the exact port of 49920 (nothing sus aboout this number)  well thats is pretty simple u just replace the 80 in the above calculation with 0 problem solved.

now back to the main quest , in active mode the sever use port 20 as source port since we are trying to connect to port 50000 as destination port so this is important here - now imagine we are trying to connect remotely , and we made this connection but my computer is behind a NAT router (NAT is a bandaid for lack of ip addresses, so NAT allows all devices in ur house to share 1 ip to communicate with enternal networks) . NOW COMES THE ISSUES - i can initiate an outbound connection to ftp server but the router firewall block unsolicited incoming connection , since active ftp needs server to initiate a seperate connection back to client the firewall might reject it resulting in ftp log in works , commands works but file donwload ,upload and listing directories cannot work.


### Passive ftp
Passive ftp changes the direction of data connection or in other words it changes who initiates the connection instead of server making a connection back to client for data connection the command connection and data connection is made by client.
1. client connect to port 21 (command control)
2. client request passive mode (PASV - passive mode)
3. server will select a data port (227 Entering Passive Mode (10,10,10,50,195,81)
4. client initiates the data connection.

#### why passive fpt is more common
the current world practices commonly use -
1. NAT
2. Stateful firewalls
3. Corporate network segmentation
4. Cloud security groups
5. Host-bacsed firewalls
So passive connection is much more less trouble to use or easier to support because client side initiates both TCP  connections but passive ftp needs the server passive port to be reachable aswell . An admin must configure the firewall to permit the necessary connection to those pasove ports

#### what does it mean to me>>>
say i found a ftp port 21 open and made a connection but i cannot download , upload data and lis directories , the probable cause to this can be  - 
1. firewall bloaking the passive port
2. incorrect passive mode configuration
3. NAT advertising an incorrect server address
4. Data connection is rejected by the client or server
5. ftp server resource or connections issues

###### EPSV - Extended passive port
unlike traditional passive n extended passive returns only the port with client using the servers existing addess which is particularly useful with ipv6 and NAT enviroments.

## FTP Authentication and session establishment

To establish a connection we can just enter ==ftp 10.10.10.10==  , this intialize a connection to ftp server and the reply would be ==220 (vsFTOd 2.0.0)== this means the server is ready for the ftp control connection , but this is not authentication.

To authenticate the client must use the command ==USER== combine with a valid username . later the server will ask for a password and enter a valid pass. After recieving the pass server will then checks the creds against the authentication mechanisms now authenticating mecanism will be based on the server itself below are some configured mechanisms  - 

- local OS accounts
- Virtual FTP account
- PAM authentication on linux (PAM is pluggable authentication module - modular linux framework that seperates user authentication task from individual apps via shared library (libpam)this allow application to support local passwords ,ldap,mfa or biometricts rather than hardcoding authentication logic into every programe, admin config stack and rules of applications.)
- Directory based authentication
If the creds are valid then you are greeted with a 230 which is successful login but if it were wrong then it will be 530 code which means well login incorrect.

### Establishing access
Yay we go into the ftp but we aint done yet now the ftp will determine what files the user have access to . They server can restrict user to a specific directory ,to implement this we can use mechanism such as -
- chroot restriction 
- filesystem permission
- ftp server access control rules 
- OS user privilage 
basically meaning u can set up RBAC .

### operations

| commands | functions                     |
| -------- | ----------------------------- |
| PWD      | print working directory       |
| LIST     | list all content in directory |
| CWD      | change working directory      |
| RETR     | return or download            |
| STOR     | upload or store a file        |

### FTP + TLS = FTPS

So the biggest issue in ftp is that it is a NON ENCRYPTED network traffic which means any teenager with wireshark that is in the same network as u can read ur files and also can get ur creds as it is sent in cleartext. Comes our saviour FTPS , so ftps add tls encryption to ftp.
 A ftps connection starts with an ftp server then later u can use ==AUTH TLS==  to get the client and server to make a tls handshake later u can use ==PBSZ 0 & PROT P== to negotiate protection for data connection. Encrypting the control connection doesnt mean data connection is protected , data connection protection must also be negotiated.

## How does anaonymous account works

Alright lets look at how anon account works on ftp , so to use anon account first the admin must enable it . Now if it is enables u can use anonymous as username and any password or even no password.

The anonymous account privilage differ based on admin config such as -
- read only
- download only
- upload access to specific directories
- no access to sensitive directories 
- restricted directory visibility
Basically it has built in RBAC. You may as what function it has well very simple imagine a office and everyone needs to download a new wallpaper instead of creating a new account users can use anon account to log in and grep that file and admin can restric anon log in to only have download functionality.

Why is this important to me well , mostly if anon ftp is allowed , it is either misconfigured or configured but it allow me to get a ftp connection into the server without cred which means it opens a door that might lead to backups , unintended directories basically imagine a house that i wanna rob and a gate is not locked there might be another locked gate behind it , there might be absolutely nothing but there might also be the key to the house to worth investigating.
Things that i find here that might be interesting are -
1. backup files - backup files may contain old creds, database connection ,internal server addr,config info,api keys,application secrets and so on .
2. public directories - public information same as above any oinformation to help understand them better and helps me dig deeper.
3. incoming directories - directories i have write access to , and if i figure out how to execute files or where or what are happening to the uploaded files i can use that to my advantage like if it leads to a webpage directory i might upload a rev shell go to the website to activate that script and catch it on my netcat for a service account.
How to check for this is very easy run , a simple nmap script 
==nmap -p21 --script ftp-anon 10.10.10.10==

 when can anon log in be vulnarable, the existance of anon account alone is not an vulnerability its a feature (like steve jobs says) but a misconfiguration of this account can lead to vulnerability.

| config                               | for what                                                            |
| ------------------------------------ | ------------------------------------------------------------------- |
| access to confidential backups       | now that is information disclosure                                  |
| access to employee records           | sensitive data exposure                                             |
| anonymous upload to isolate storage  | unauthorised storage risk , can be a very big issue                 |
| upload into web executtion directory | severe security issue potentiolly can get access to service account |
## What information can a Pentester get from FTP
Useful information attacker can get from ftp . 
1. service identification
2. service version
3. ftp banner
4.  OS information
5. directory and file information
6. user and permission information
7. authentication information
8. supported ftp feature

okay lets go 1 by 1 on how we collect these information
1.  to get service identification and service version we can use namp . this is 1 of the simplest ways simply use -sV in nmap scan. Why it matters well - different service or implementation have different configuration, authentication mechanism , security feature , known vulnerabilities and historical issue or exsisting CVEs.
2. banner grabbing can also be useful , another way to grab ftp software , software version , server identity, custom admin messeges, additional implementation messeges if lucky. This can give enough information to narrow down possible attack surface or help discover existing vulnerabilites. banner grab with ==nc -nvlp 10.10.10.10. 21==
3. OS information , sometimes ftp servers can provide more detailed information about the os which helps us. can be done with nmap script  ==--script ftp-syst==
4. directory information can help identify current directory , directory structure, available files, file size,file permission when listing format suppots them,sensitive directories if lucky. can be done with ==PWD , LIST, NLST , CWD public== all thru anon login or with creds.
5. user and file permission can be discovered based on our accessebility and server configuration. ftp usernames, file ownership,group ownership,file permission, whether files are readable , directories writable ,uploads are allowed or not.
6. authentication information can give information whether anon login in permitted , authentication is must or no, how the server respond to invalids,whether it support tls , whether the services exposes user enumeration behavior.
7. Now lets look at supported feature, u can see this when u connecct to ftp and use the command ==FEAT== . Okay why is this useful well it exposes supported trasfer modes, extended passive modes , file metadata operations , restart capabilities and other useful features . We can also use ==STAT== command to ger current connection status , session infor, server software , transfer status and whether the session is authenticated or not.

So as a pentester i can get a better understanding oof the attack surface rather than port 21 open (uuuuu i see). the example below give a much clearer picture , all the information u need to figure out a way into ftp if there is any (self note - i should really create a checklist for all the common ports during checking for starting out to not miss out any info that might help).Example -

| Information        | findings     |
| ------------------ | ------------ |
| port               | 21           |
| Software           | vsftpd       |
| version            | 3.0.0        |
| anonymous access   | enabled      |
| root directory     | /            |
| sensitive files    | backup.sql   |
| writable directory | /incoming    |
| TLS                | not required |
| data mode          | passive      |

## FTP misconfiguration

Simple ftp misconfiguration can pose a severe security risk , just because the functions as it should doesnt mean it is secure . As simple as havking anonymous account permission to backups can leak sensitive information to attacker.

### what to look for (uu fun part)

|     | what to look for                                      | why                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| --- | ----------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | anonymous access enabled unnecessarily                | This is important , if a organization want only employees to access the data thru ftp this could cause -<br>1. unauthorized file access<br>2. sensitive information disclosure<br>3. unauthorized uploads<br>4. information that is useful for further attacks                                                                                                                                                                                              |
| 2   | Writable directories exposed to unauthenticated users | A writable directory can potentioly cause <br>1. unauthorised file uploads<br>2. hosting malicious files<br>3. storage abuse<br>4. file replacement or dedletion depending on permission (danger to availability and authenticity)<br>5.Can also compromise directories that is connected to another application                                                                                                                                            |
| 3   | Cleartext authentication                              | We can check for tls support with ==--script ssl-enum-cipher== or ==openssl s_client -starttls ftp -connect target:21== this helps to check <br>1. AUTH TLS is supported<br>2. whether tls is mandotory<br>3. whether client can fallback to plaintext<br>4. whether the certificate is valid<br>5. whether data connection is also protected                                                                                                               |
| 4   | Excessive file permission                             | say ftp account can read etc/passwd or etc/shadow , these are unnecessary access , if an attacker get access to ftp account with permission that allow then to broad filesystem access they can use that to attack the server further                                                                                                                                                                                                                       |
| 5   | FTP directory exposed thru web server                 | say the attacker can access www.example.com/upload thru the browser and anonymous upload are enabled then an attacker can simply upload a rev shell to get access , this pave the path to remote code execution.                                                                                                                                                                                                                                            |
| 6   | weak authentication controls                          | we should also be vary of <br>1. weak password<br>2. unrestricted authentication attempts<br>3. Shared account<br>4. default creds remain enabled<br>5. failed login attempts that are not monitored<br>6.account lockout or rate limiting if configured appropriately                                                                                                                                                                                      |
| 7   | outdated ftp software                                 | mostly every software update will introduce a fix for existing bug, cve, vulnerability and so on. To have a updated software is important while we think dont fix if it aint broke sometime inherent risk can be higher than expected if we dont take into account for chained attaks                                                                                                                                                                       |
| 8   | Excessive network exposure                            | say if a company needs only internal network employee to access the ftp but the firewall allow any ip or users from external network this increases the attack surface. investigate<br>1. whether ftp needs to be internet accessible<br>2. whether source ip restiction exist<br>3. whether admin access is exposed<br>4.whether passive data ports are restricted to require range<br>5. whether ftp access is segmented from sensitive internal networks |

