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

Open a browser on Attacker Desktop, navigate to https://mail.cyberbotic.io and login with the obtained credentials.












