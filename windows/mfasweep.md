### Source
https://raw.githubusercontent.com/dafthack/MFASweep/master/MFASweep.ps1  

### Check if mfa is enabled for:
```
Microsoft Graph API - exploit using Import-Module MSOnline; Connect-MsolService or $credential = Get-Credential; Connect-MsolService -Credential $credential
Azure Service Managment API - exploit using Import-Module Az; Connect-AzAccount or $credential = Get-Credential; Connect-AzAccount -Credential $credential
Microsoft 365 Exchange Web Services - exploit using (mailsniper) Invoke-SelfSearch -Mailbox <user>@<domain> -ExchHostname outlook.office365.com -Remote
Microsoft 365 Web Portal  - exploit using browser https://outlook.office365.com or https://portal.azure.com
Microsoft 365 Web Portal /w Mobile User Agent - exploit by changing the user agent string https://login.microsoftonline.com
Microsoft 365 ActiveSync - exploit using build in mail application in windows
```

### Check mfa settings without trigger MFA request
```
Invoke-MFASweep -Username <user>@<domain> -Password <password>
```

