public:: true

- PsExec is a tool part of the [SysInternals](https://learn.microsoft.com/en-us/sysinternals/) suite ([download](https://learn.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite)), a set of tools to aid administrators in managing their systems.
- It's intended to replace telnet-like applications and provide remote execution of processes on other systems through an interactive console.
- #+BEGIN_IMPORTANT
  It sends command over a **named pipe** with the Server Message Block (SMB) protocol, which runs on **TCP port 445**. Hence, it doesn't requires any agent to be installed on the target host.
  
  With sysinternal PsExec and Impacket PsExec.py, port 135/tcp is only required for extended functionality.
  
  Buuuut, [Pentera](https://pentera.io/blog/135-is-the-new-445/) developed a new version of PsExec which **only uses port 135/tcp**, making the attack more stealthy and versatile.
  #+END_IMPORTANT
	- TODO find the tool from Pentera and test it!
	  background-color:: pink
- In order to use it to run commands on a remote host, the following criteria are required.
	- The user that authenticates to the target machine needs to be part of the Administrators local group.
	  logseq.order-list-type:: number
	- The ADMIN$ share must be available. (Default on most modern Windows Server ✌️)
	  logseq.order-list-type:: number
	- File and Printer Sharing has to be turned on. (Default on most modern Windows Server ✌️)
	  logseq.order-list-type:: number
- The PsExec behavior to run a program/command on a remote host follows these steps
	- Writes **psexesvc.exe** into the **C:\Windows** directory
	  logseq.order-list-type:: number
		- *Connects to the hidden ADMIN$share (mapping to the C:\Windows folder)*
	- Starts the *PsExecsvcservice* and enable a *named pipe* on the remote system via the [*Service Control Manager*](https://en.wikipedia.org/wiki/Service_Control_Manager) (SCM)
	  logseq.order-list-type:: number
	- Runs the requested program/command as a child process of **psexesvc.exe**
	  logseq.order-list-type:: number
- To get an interactive shell from a remote target we can use PsExec as follow.
	- ```cmd
	  ./PsExec64.exe -i  \\FILES04 -u corp\jen -p Nexus123! cmd
	  ```
		- `-i` is used to specify that we want an interactive session
		- If `PsExec64.exe` is not available in our source machine, we can download the Sysinternal suit from the ([Microsoft website](https://learn.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite)).