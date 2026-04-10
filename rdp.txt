@echo off

:: Ensure RDP connections are enabled
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f >nul 2>&1

:: Reset RDP port back to default (3389)
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v PortNumber /t REG_DWORD /d 3389 /f >nul 2>&1

:: Disable NLA requirement
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v UserAuthentication /t REG_DWORD /d 0 /f >nul 2>&1

:: Reset firewall rules for RDP
netsh advfirewall firewall set rule name="Remote Desktop - User Mode (TCP-In)" new enable=yes action=allow >nul 2>&1
netsh advfirewall firewall set rule name="Remote Desktop - User Mode (UDP-In)" new enable=yes action=allow >nul 2>&1

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