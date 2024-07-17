```bash
@echo off
set VMName=BRG
 
echo Restarting VM %VMName% ...
PowerShell -NoProfile -Command "Restart-VM -Name '%VMName%' -Force"
 
echo VM %VMName% has been restarted.
exit
```

>  BRG为虚拟机名称，可根据实际改为需要重启的虚拟机名称