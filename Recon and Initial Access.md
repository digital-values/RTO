*Reconnaissance*

There are two main facets of recon - organisational and technical.

During organisational recon, you’re focused on collecting information about the organisation.  
This can include the people who work there (names, jobs and skills), the organisational structure, site locations and business relationships.

During technical recon, you’re looking for systems such as public-facing websites, mail servers, remote access solutions, and any vendors or products in use, 
particularly defensive ones - web proxies, email gateways, firewalls, antivirus etc.

Gathering either type of information can be done passively or actively.

Tools: whois, dig, dnscan, google dorking, SOCMINT

#PASSWORD SPRAYING 
Password spraying is an effective technique for discovering weak passwords that users are notorious for using. 
Patterns such as MonthYear (August2019), SeasonYear (Summer2019) and DayDate (Tuesday6) are very common.
- Be cautious of localisations, e.g. Autumn vs Fall.

Two excellent tools for password spraying against Office 365 and Exchange are MailSniper and SprayingToolkit. 

1. Enumerate the NetBIOS name of the target domain with Invoke-DomainHarvestOWA.
