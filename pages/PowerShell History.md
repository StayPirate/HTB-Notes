public:: true

- Since PowerShell is a vital resource for attackers, as a defensive measure IT staff started collecting and recording information about executed commands in PowerShell. This is done to review and respond accordingly to security threats.
- By default Windows only logs a small amount of information on PowerShell usage, but there are a couple of mechanisms which enable more verbose logging.
	- PowerShell [Transcription](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.host/start-transcript?view=powershell-7.4)
		- AKA *over-the-shoulder-transcription* because the logged information is equal to what a person would obtain from looking over the shoulder of a user entering commands in PowerShell.
		- Information is stored in *transcript files*, which are often saved in
			- The user's home directorie
			- A central directory for all users of a machine
			- A network share collecting the files from all configured machines
	- PowerShell [Script Block Logging](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_logging_windows?view=powershell-7.2)
		- A much broader logging as it records commands and blocks of script code as events while executing
- #+BEGIN_WARNING
  While this logging mechanisms are great from the defensive perspective, they often **contain valuable information** [for attackers](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_logging_windows?view=powershell-7.4#protected-event-logging).
  #+END_WARNING
- PowerShell History
	- First, always check the PowerShell history of a user. We may be lucky and find sensitive information there.
	- To browse the history we can use the [`Get-History`](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/get-history?view=powershell-7.4) cmdlet in PowerShell.
	- Retrieve **deleted history**
		- The cmdlet [`Clear-History`](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/clear-history?view=powershell-7.2) is sometimes used by administrators to clean the PowerShell history, but this cmdlet only clear the history that can be retrieved with `Get-History`.
		- Starting with PowerShell v5, v5.1, and v7, a module named [PSReadline](https://learn.microsoft.com/en-us/powershell/module/psreadline/?view=powershell-7.2) is included, which is used for line-editing and command history functionality.
		- #+BEGIN_IMPORTANT
		  `Clear-History` **does not clear** the command history recorded by PSReadline.
		  #+END_IMPORTANT
		- To find on which file PSReadline stores its PowerShell history we can use [`Get-PSReadlineOption`](https://learn.microsoft.com/en-us/powershell/module/psreadline/get-psreadlineoption?view=powershell-7.4)
		  ```powershell
		  PS C:\Users\dave> (Get-PSReadlineOption).HistorySavePath
		  (Get-PSReadlineOption).HistorySavePath
		  C:\Users\dave\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
		  ```
			- #+BEGIN_PINNED
			  Sometimes other history files might be stored on that folder, so ensure to check anything inside it.
			  #+END_PINNED
		- We can now review the fully uncleared history by reading that file
		  ```powershell
		  PS C:\Users\dave> cat C:\Users\dave\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
		  ...
		  $PSVersionTable
		  Register-SecretVault -Name pwmanager -ModuleName SecretManagement.keepass -VaultParameters $VaultParams
		  Set-Secret -Name "Server02 Admin PW" -Secret "paperEarMonitor33@" -Vault pwmanager
		  cd C:\
		  ```
- PowerShell Transcription
	- By default transcripts are stored in `$HOME\Documents`, but that could be customized by the administrator.
	- We can try to locate transcripts with [`Get-ChildItem`](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-childitem?view=powershell-7.4) then read them from a terminal
	  ```powershell
	  PS C:\Users\dave> Get-ChildItem -Path C:\ -Include *transcript* -File -Recurse -ErrorAction SilentlyContinue
	  ```
	  ```powershell
	  PS C:\Users\dave> type C:\Users\Public\Transcripts\transcript01.txt
	  type C:\Users\Public\Transcripts\transcript01.txt
	  **********************
	  Windows PowerShell transcript start
	  Start time: 20220623081143
	  Username: CLIENTWK220\dave
	  RunAs User: CLIENTWK220\dave
	  Configuration Name: 
	  Machine: CLIENTWK220 (Microsoft Windows NT 10.0.22000.0)
	  Host Application: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
	  Process ID: 10336
	  PSVersion: 5.1.22000.282
	  ...
	  **********************
	  Transcript started, output file is C:\Users\Public\Transcripts\transcript01.txt
	  PS C:\Users\dave> $password = ConvertTo-SecureString "qwertqwertqwert123!!" -AsPlainText -Force
	  PS C:\Users\dave> $cred = New-Object System.Management.Automation.PSCredential("daveadmin", $password)
	  PS C:\Users\dave> Enter-PSSession -ComputerName CLIENTWK220 -Credential $cred
	  PS C:\Users\dave> Stop-Transcript
	  **********************
	  Windows PowerShell transcript end
	  End time: 20220623081221
	  **********************
	  ```
- PowerShell Script Block Logging
	- Event logs might be encrypted with asymmetric cryptography *[(Protected Event Logging)](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_logging_windows?view=powershell-7.4#protected-event-logging)* to avoid leaking sensitive information. This requires administrator to deploy the public key on each machine, so may be difficult to find such a configuration.
	- TODO find how to read event logs generated by the *Script Block Logging*, if available in the local machine.
	  background-color:: pink