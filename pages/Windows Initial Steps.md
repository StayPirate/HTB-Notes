public:: true
Tags:: powershell, windows,grep

- #### Check current user privileges
  logseq.order-list-type:: number
  
  ```powershell
  whoami /priv
  ```
- #### Gather some information on our target system
  logseq.order-list-type:: number
  
  ```powershell
  systeminfo | findstr /B /C:"Domain" /C:"Host Name" /C:"OS Name" /C:"OS Version" /C:"System Type" /C:"Hotfix(s)"
  ```
- #### Check for stored credentials on the system
  logseq.order-list-type:: number
  
  ```powershell
  cmdkey /list
  ```
- #### Check if current user is part of any interesting groups
  logseq.order-list-type:: number
  
  ```powershell
  net user <username>
  net user [Environment]::UserName # Need to be tested
  ```
- #### Check who's part of the local administrators group
  logseq.order-list-type:: number
  ```powershell
  net localgroup administrators
  ```
- Get a list of domain user accounts with associated SPNs ([[Kerberoasting]])
  logseq.order-list-type:: number
  {{embed ((65684a10-0764-4ca6-ad8e-1053e99ded78))}}
- #### Hunt for any non-standard folders and files
  logseq.order-list-type:: number
	- Search in `C:\`
	  
	  ```powershell
	  cmd.exe /c dir /a C:\
	  ```
	- Search in `C:\Programs Files`
	  ```powershell
	  cmd.exe /c dir /a C:\Program........
	  ```
		- TODO Search in both Program Files directories
- logseq.order-list-type:: number
	- `icacls` (built-in)
	  logseq.order-list-type:: number
	  
	  ```powershell
	  icacls "C:\Custom Tasks\Backup"				# Example folder
	  icacls "C:\Custom Tasks\Backup\tftp.exe"	# Example file
	  ```
	- `accesschk` (from the Sysinternals Suite)
	  logseq.order-list-type:: number
	  *If you don’t have the Sysinternals Suite on your attacker machine, you can grab it [here](https://docs.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite).*
	  
	  ```powershell
	  # Example folder
	  .\accesschk64.exe -wvud "C:\Custom Tasks\Backup" -accepteula
	  # Example file
	  .\accesschk64.exe -wvu "C:\Custom Tasks\Backup\tftp.exe" -accepteula
	  ```
	  The ‘-d’ switch checks permissions of the folder itself. Without `-d` it checks permissions of all the files inside the folder.
- #### Search for interesting folders or files
  logseq.order-list-type:: number
	- Recursively search in the `C:\Users` folder to see if we are able to list any folders/files in other users profile folders
	  logseq.order-list-type:: number
	  
	  ```powershell
	  # Shwo only non-hidden files
	  Get-ChildItem -Recurse C:\users | Select FullName
	  # Show hidden and non-hiddend files
	  Get-ChildItem -Recurse -Force C:\users | Select FullName
	  ```
	  NB: In *powershell* `dir` is an alias of `Get-ChildItem`.
	- Recursively search for .txt files in the current folder
	  logseq.order-list-type:: number
	  
	  ```powershell
	  # Show non-hiddend txt files
	  Get-ChildItem . -Filter "*.txt" -Recurse
	  # Show hidden and non-hiddend txt files
	  Get-ChildItem -Recurse -Force -Filter "*.txt" C:\users | Select FullName
	  ```
	- Recursively search files with more tha 100 lines
	  logseq.order-list-type:: number
	  
	  ```powershell
	  dir . -filter "*.txt" -Recurse | ? {(gc $_.FullName | Measure -Line | Select -Expand Lines) -gt 100 }
	  ```
	- Get a list of all files in the current directory that match the specified pattern and ignore errors
	  logseq.order-list-type:: number
	  
	  ```powershell
	  Get-ChildItem -Include *.doc,*.docx -File -Recurse -ErrorAction SilentlyContinue
	  ```
	- Grepping
	  logseq.order-list-type:: number
		- For file name
		  logseq.order-list-type:: number
		  ```powershell
		  Get-ChildItem -Recurse | Where {$_.Name -match 'Replace'} | Select Fullname
		  Get-ChildItem | Where-Object { $_.Name -match '[a-z].txt$' }
		  ```
		- For folder name
		  logseq.order-list-type:: number
		  
		  ```powershell
		  Get-ChildItem -Recurse | Where {$_.DirectoryName -match 'Debug'} | Select Fullname
		  ```
	- Find files older than a specific date
	  logseq.order-list-type:: number
	  
	  ```powershell
	  Get-ChildItem -File | Where-Object {$_.CreationTime -lt (Get-Date).AddDays(-15)} | Select Name, CreationTime | sort CreationTime -Descending
	  ```