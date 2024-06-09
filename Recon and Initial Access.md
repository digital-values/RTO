# Reconnaissance

There are two main facets of recon - organisational and technical.

During organisational recon, you’re focused on collecting information about the organisation.  
This can include the people who work there (names, jobs and skills), the organisational structure, site locations and business relationships.

During technical recon, you’re looking for systems such as public-facing websites, mail servers, remote access solutions, and any vendors or products in use, 
particularly defensive ones - web proxies, email gateways, firewalls, antivirus etc.

Gathering either type of information can be done passively or actively.

Tools: whois, dig, dnscan, google dorking, SOCMINT

### PASSWORD SPRAYING 
Password spraying is an effective technique for discovering weak passwords that users are notorious for using. 
Patterns such as MonthYear (August2019), SeasonYear (Summer2019) and DayDate (Tuesday6) are very common.
- Be cautious of localisations, e.g. Autumn vs Fall.

Two excellent tools for password spraying against Office 365 and Exchange are MailSniper and SprayingToolkit. 

1. Enumerate the NetBIOS name of the target domain with Invoke-DomainHarvestOWA.
2. Next, we need to find valid usernames from the list of users enumerated from https://cyberbotic.io.
3. namemash.py is a python script that I've used for as long as I can remember. It will take a person's full name and transform it into possible username permutations.
4. Invoke-UsernameHarvestOWA uses a timing attack to validate which (if any) of these usernames are valid.
5. MailSniper can spray passwords against the valid account(s) identified using, Outlook Web Access (OWA), Exchange Web Services (EWS) and Exchange ActiveSync (EAS).
6. We can do further actions using MailSniper with valid credentials, such as downloading the global address list.
7. If there are names here that we didn't find during initial recon, we can go back and do another round of spraying against them.

### Internal Phishing 

Open a browser on Attacker Desktop, navigate to https://mail.cyberbotic.io and login with the obtained credentials.

#### Initial Access Payloads 


Sending a payload to the phished user(s) is a direct way to gain a foothold, since it will be executed on their system.  There are broadly two options for delivering a payload.

    Send a URL where a payload can be downloaded.
    Attach the payload to the phishing email.

Reading Mark of The Web (MOTW) with PowerShell:

PS C:\Users\bfarmer\Downloads> gc .\test.txt -Stream Zone.Identifier

The possible zones are:

    0 => Local computer
    1 => Local intranet
    2 => Trusted sites
    3 => Internet
    4 => Restricted sites

Files that are emailed "internally" via a compromised Exchange mailbox are not tagged with a Zone Identifier.


##### VBA Macros 

Instructions to re-arm Office license
- Run a Command Prompt as a local admin
-  cd C:\Program Files\Microsoft Office\Office16
- cscript ospp.vbs /rearm

Basic Shell:

Sub AutoOpen()

  Dim Shell As Object
  Set Shell = CreateObject("wscript.shell")
  Shell.Run "notepad"

End Sub

"wscript" is the Windows Script Host, which is designed for automation.  The "shell" method provides the ability to execute OS commands.  To test the above code, use the play/pause/stop buttons.

add to macros, change the author and save as a ".doc" 97-03 document best compromise method 

We then want to upload this file to the team server as well.  Go to Site Management > Host File and select your document.

Send the document to the "co-worker" of the compromised host. 

Pray that the "co-worker" opens the document and enables macros.

http beacon will appear

##### Remote Template Injection 

using a remote template injection will allow you to save your payload as a template and modify it to send as a ".docx" 

1. Open a new blank word document and insert whatever VBA Macro payload you want
   1b. Save the template as a 97-2003 ".dot" in your payloads drive
2. host the file on your cobalt strike web management server or web server
   2b. host it on www.yourbaddomain.com/yourmalicioustemplate.dot
3. Open a new custom office template from "Documents/Custom Office Template/Blank Template"
   3b. add whatever words and save it as a "docx"
4. Send it from your compromised target to internal members
5. Beacon



Some organisations (particularly those with an internal PKI) will perform SSL offloading on HTTPS web traffic.  This allows the proxy to decrypt incoming HTTPS traffic and inspect the plaintext HTTP.  The traffic is then re-encrypted with an internally trusted private key before being forwarded to the client.

This means that even your HTTPS C2 traffic can be inspected.  Some C2 tools (such as Covenant) allow you to configure certificate pinning on the implants which would effectively prevent this from taking place, but at the potential cost of the proxy blocking the traffic entirely.

This may go without saying - but if a web proxy has the ability to read and inspect HTTP(S) traffic, then it can also scan for known malicious content.  However, this does come with a performance penalty.  Another common feature is to block the download and/or upload of particular file types, such as .exe, .dll, .ps1, etc, which may impact your ability to deliver payloads.

Many organisations will also require a form of authentication before a client is allowed use a proxy.  This could be anything from basic auth with a local database, Radius or Active Directory integration.  AD integration is a very common and popular choice as it provides single sign-on via NTLM, Kerberos and LDAP(S).

This often means that a principal has to be in a "trusted" domain group before they can use the proxy, such Domain Users or a custom group entirely.  This does often exclude computer accounts, which means HTTP(S) Beacons running as local SYSTEM accounts cannot work.


##### HTML Smuggling 

HTML smuggling is a technique that uses JavaScript to hide files from content filters.  If you send a phishing email with a download link, the HTML may look something like:

* href="http://attacker.com/file.doc">Download Me *

Email and web scanners are capable of parsing these out and taking some action.  They may be removed entirely, or the URL content fetched and scanned by an AV sandbox.  HTML smuggling allows us to get around this by embedding the payload into the HTML source code and using JavaScript to construct URLs by the browser at runtime.




















