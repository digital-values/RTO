This section will review some of the information you can enumerate from the current domain as a standard domain user. We'll cover many of these areas (e.g. domain trusts and GPO abuses) in much more detail when we get to those specific chapters. For now, we'll see some of the different tooling that can be used to query the domain, and how we can obtain targeted information.
It's worth noting that performing domain recon in a high integrity process is not required, and in some cases (such as SYSTEM) can be detrimental.

# PowerView 

PowerView has long been the de-facto tool for domain enumeration.  One of its biggest strengths is that the queries return proper PowerShell objects, which can be piped to other cmdlets.  This allows you to chain multiple commands together to form complex and powerful queries.

beacon> powershell-import C:\Tools\PowerSploit\Recon\PowerView.ps1

### Get-Domain

Returns a domain object for the current domain or the domain specified with -Domain. Useful information includes the domain name, the forest name and the domain controllers.

beacon> powershell Get-Domain

### Get-DomainController

Returns the domain controllers for the current or specified domain.

  beacon> powershell Get-DomainController | select Forest, Name, OSVersion | fl

### Get-ForestDomain

Returns all domains for the current forest or the forest specified by -Forest.

beacon> powershell Get-ForestDomain
