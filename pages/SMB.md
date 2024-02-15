public:: true
alias:: samba

- Null Session
	- TODO description
	- With [[CrackMapExec]]
	  ```bash
	  crackmapexec smb 10.0.2.8 -u ‘’ -p ‘’
	  ```
	- With ...
		- This tool attempts to establish a null session with the target and then uses RPC to extract useful information.
		  ```bash
		  enum4linux -a 10.0.2.8
		  ```
			- Listing of file shares and printers
			- Domain/Workgroup information
			- Password policy information
			- RID cycling output to enumerate users and groups
			- More ...
- Guest user
	- If we didn't get any results with Null Session, then we can try to access with the well-known *Guest* user.
	- With [[CrackMapExec]]
	  ```bash
	  crackmapexec smb 10.0.2.8 -u ‘guest’ -p ‘’
	  ```
- List shares
	- Once we have a way to authenticate via SMB we can list available shares.
	- With [[CrackMapExec]]
	  ```bash
	  crackmapexec smb 10.0.2.8 -u ‘guest’ -p ‘’ --shares
	  ```
- Enumerate resources via `IPC$`
	- In case we get **READ** access to the *Remote IPC* (AKA `IPC$`), then we can enumerate **some** *local* and *domain* resources via SID looksup.
	- With [[CrackMapExec]]
	  ```bash
	  crackmapexec smb 10.0.2.8 -u ‘guest’ -p ‘’ --rid-brute
	  ```
		- Please replace `-u` and `-p` with the creds that gives you read access to the *Remote IPC*.
	- With [[Impacket-lookupsid]]
	  ```bash
	  impacket-lookupsid guest@10.0.2.8
	  ```
		- Remind to use the creds that gives you read access to the *Remote IPC*.
		- Target is defined as `[[domain/]username[:password]@]<targetName or address>`