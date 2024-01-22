public:: true
alias:: MIC

- *Mandatory Integrity Control* (MIC) is a core security feature that adds mandatory access control to running processes based on their *[Integrity Level](https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation/integrity-levels)* (IL).
- Integrity Levels are *measurements of trust*.
- It's implemented in Windows Vista and later.
- The IL for a [Security Principals](((65ae443c-68a0-4b2e-b534-b7cf8934ec14))) is defined i its [access token]([[AD Access Tokens]]) when initialized.
- The IL for an object (processes, files, and other securable objects) is defined in its *[security](https://learn.microsoft.com/en-us/windows/win32/secauthz/security-descriptors) [descriptor](https://en.wikipedia.org/wiki/Security_descriptor)*.
- When a principal (e.g. a user) tries to access an object, the *[Security Reference Monitor](https://learn.microsoft.com/en-us/windows-hardware/drivers/kernel/windows-kernel-mode-security-reference-monitor)* compares its IL against the IL of the object. Then it will allow or not allow access to such a resource.
	- For example, a principal with a low integrity level cannot access an object with a medium integrity level.
- Windows defined Integrity Levels
	- Untrusted
		- *Lowest integrity level with extremely limited access rights for processes or objects that pose the most potential risk*
	- Low
		- *Very restricted rights often used in sandboxed*
	- Medium
		- *Standard users*
		- *Objects that lack an integrity label are treated as medium to prevent low-integrity code from modifying unlabeled objects*
	- High
		- *Elevated users*
	- System
		- *System services and kernel*
- When a principal starts a process or creates an object, this will receive the principal IL.
	- In case the executable file's IL is set to low, then the process will receives a low IL regardless the IL of the pricipal who executed it.
- Show current Integrity Levels
	- For processes this can be viewed with [Process Explorer](https://learn.microsoft.com/en-us/sysinternals/downloads/process-explorer) (part of sysinternals)
	  ![image.png](../assets/image_1705939536629_0.png)
	- For the current user this can be found via `whoami /groups`
	  ```cmd
	  whoami /groups
	  
	  GROUP INFORMATION
	  -----------------
	  
	  Group Name
	  ================================================
	  Mandatory Lable\Medium Mandatory Level
	  ```
	- For files this can be viewed vith [`icacls`](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/icacls)
	  ```cmd
	  icacls test.txt
	  test.txt BUILTIN\Administrators:(I)(F)
	           DESKTOP-IDJHTKP\user:(I)(F)
	           NT AUTHORITY\SYSTEM:(I)(F)
	           NT AUTHORITY\INTERACTIVE:(I)(M,DC)
	           NT AUTHORITY\SERVICE:(I)(M,DC)
	           NT AUTHORITY\BATCH:(I)(M,DC)
	           Mandatory Label\High Mandatory Level:(NW)
	  ```