@echo off

:: Ensure RDP connections are enabled
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f >nul 2>&1

:: Reset RDP port to 50000
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v PortNumber /t REG_DWORD /d 50000 /f >nul 2>&1

:: Reset firewall rules for RDP on port 50000
netsh advfirewall firewall add rule name="Remote Desktop - User Mode (TCP-In)" protocol=TCP dir=in localport=50000 action=allow enable=yes >nul 2>&1
netsh advfirewall firewall add rule name="Remote Desktop - User Mode (UDP-In)" protocol=UDP dir=in localport=50000 action=allow enable=yes >nul 2>&1

:: Ensure services are set to auto-start
sc config TermService start= auto >nul 2>&1
sc config UmRdpService start= auto >nul 2>&1

:: Ensure dependent service is running
net start UmRdpService >nul 2>&1

:: Restart RDP service
sc query TermService | find "RUNNING" >nul 2>&1
if %errorlevel% neq 0 (
    net start TermService >nul 2>&1
)