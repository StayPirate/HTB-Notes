public:: true

- [PowerView](https://powersploit.readthedocs.io/en/latest/Recon/#powerview) is a PowerShell tool to gain network situational awareness on Windows domains. It contains a set of pure-PowerShell replacements for various windows "net *" commands, which utilize PowerShell AD hooks and underlying Win32 API functions to perform useful Windows domain functionality.
- It can also:
	- Identify where on the network specific users are logged into.
	- Check which machines on the domain the current user has local administrator access on.
	- Enumerate and abuse domain trusts.
- Source code available [here](https://github.com/PowerShellMafia/PowerSploit/raw/master/Recon/PowerView.ps1). *(GH repository archived on Jan 21, 2021)*
- HackTricks docs [here](https://book.hacktricks.xyz/windows-hardening/basic-powershell-for-pentesters/powerview).
- [SharpView](https://github.com/tevora-threat/SharpView) is a .NET port of PowerView. *(Last update was in 2018)*
- Enumeration
	- Log to the *target machine* as a domain user.
	  #+BEGIN_NOTE
	  Local users cannot query the Domain Controller.
	  #+END_NOTE
	- Upload/Download `PowerView.ps1` to a writable directory (e.g. `C:\Temp`)
	- Get a PowerShell terminal and load it in-memory
	  ```powershell
	  Import-Module .\PowerView.ps1
	  ```
	- Domain
		- Retrieve basic information about the domain
		  ```powershell
		  Get-NetDomain
		  ```
	- Users and Groups
		- Retrieve the list of all users in the domain
		  ```powershell
		  Get-NetUser | select cn
		  ```
			- Retrieve groups for a specific user
			  ```powershell
			  Get-NetGroup -UserName "joe"
			  ```
		- Retrieve the list of all groups in the domain
		  *[Here](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-groups#default-active-directory-security-groups) the list of the default AD groups. Useful to quickly spot custom domain groups.*
		  ```powershell
		  Get-NetGroup | select cn
		  ```
			- Retrieve members of a specific group.
			  ```powershell
			  Get-NetGroup "Sales Department" | select member
			  ```
			  *(This include unresolved [nested groups](((6559cd1a-af38-44ff-b7ba-396d3c250b67))))*
			- Retrieve a complete list of members of a specific group.
			  ```powershell
			  Get-NetGroupMember -Identity "Sales Department" -Recurse
			  ```
			  *(The `-Recurse` option will print the users inside [nested groups](((6559cd1a-af38-44ff-b7ba-396d3c250b67))))*
	- Computers
		- Retrieve the list of all domain joined computers
		  ```powershell
		  Get-NetComputer | select dnshostname,operatingsystem,operatingsystemversion
		  ```
	- Permissions and Logged on Users
		- *Spraying* the environment to find possible *local administrative access* on computers under the current
		  user context
		  ```powershell
		  Find-LocalAdminAccess
		  ```
		  This command scans the network in an attempt to determine if our current user has administrative permissions on any computers in the domain.
		  The command relies on the [*OpenServiceW function*](https://portal.offsec.com/courses/pen-200/books-and-videos/modal/modules/active-directory-introduction-and-enumeration/manual-enumeration-expanding-our-repertoire/getting-an-overview-permissions-and-logged-on-users#fn1), which will connect to the *Service Control Manager* (SCM) on the target machines. The SCM essentially maintains a database of installed services and drivers on Windows computers. PowerView will attempt to open this database with the *SC_MANAGER_ALL_ACCESS* access right, which require administrative privileges, and if the connection is successful, PowerView will deem that our current user has administrative privileges on the target machine.
		- Retrieve logged users in other computers
			- With PowerView
				- It provides the `Get-NetSession` command which uses the [`NetWkstaUserEnum`](https://learn.microsoft.com/en-us/windows/win32/api/lmwksta/nf-lmwksta-netwkstauserenum) and [`NetSessionEnum`](https://learn.microsoft.com/en-us/windows/win32/api/lmshare/nf-lmshare-netsessionenum) APIs under the hood. The former requires administrative privileges, while the latter does not.
				  ```powershell
				  Get-NetSession -ComputerName files04 -Verbose
				  ```
				  `NetSessionEnum` got restricted since the following Windows versions, so it might no longer useful for this purpose.
					- Since Windows Server 2019 build 1809
					- Since Windows 10 build 1709
			- With [PsLoggedOn](https://learn.microsoft.com/en-us/sysinternals/downloads/psloggedon), part of the [SysInternals Suite](https://learn.microsoft.com/en-us/sysinternals/). (*Download [here](https://download.sysinternals.com/files/PSTools.zip)*)
			  *An applet that displays both the locally logged on users and users logged on via resources for either the local computer, or a remote one.*
				- #+BEGIN_NOTE
				  PsLoggedOn relies on the *Remote Registry service* in order to scan the associated key.
				  
				  The *Remote Registry service* has not been enabled by default on Windows workstations since Windows 8, but system administrators may enable it.
				  
				  It is enabled by default on Windows Server 2012 R2, 2016 (1607), 2019 (1809), and 2022 (21H2).
				  #+END_NOTE
				- List users logged on specific computer
				  ```powershell
				  .\PsLoggedon.exe \\files04
				  ```
				- List all the computer where a specific user is loggeed
				  ```powershell
				  .\PsLoggedon.exe CORP\joe
				  ```
				  *Unfortunately, there are [many reports](https://www.reddit.com/r/sysadmin/comments/p4fffi/psloggedon_question/) [over the internet](https://4sysops.com/archives/psloggedon-view-logged-on-users-in-windows/#rtoc-3) showing that this last command* ***doesn't work most of the time***. *[Here](https://www.waynehoggett.com/2011/03/psloggedon-getting-started-on-windows.html) a use-case where it might work.*
				- #+BEGIN_NOTE
				  Note that PsLoggedOn will show you as logged on via resource share to remote computers that you query because a logon is required for PsLoggedOn to access the Registry of a remote system.
				  #+END_NOTE
		- Service Accounts
		  id:: 6559f91b-875c-4d67-b318-ebc7478a9d7a
		  *We want to check them because they might be members of [high-privileged groups](https://learn.microsoft.com/en-us/entra/architecture/service-accounts-on-premises).*
			- When applications like *Exchange*, *MSSQL*, or *IIS* are integrated into AD, a unique service instance identifier known as **Service Principal Name (SPN)** associates a service to a specific *service account* in Active Directory.
			- Since these types of accounts are used to run services, we can assume that they have more privileges than regular domain user accounts.
			- Enumerate SPNs in the domain
			  *No need to run a broad port scan, simply enumerating all SPNs in the domain by querying the DC.*
				- With [legacy Windows' tools](((6559cc20-1bf4-4d59-a498-f183b3ae7fc6)))
				- With PowerView
				  ```powershell
				  Get-NetUser -SPN | select samaccountname,serviceprincipalname
				  ```