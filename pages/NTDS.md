public:: true
alias:: NTDS.dit

- The **NTDS.dit** file is the core of [[Active Directory]] and stands for *New Technology Directory Services - Directory Information Tree*.
- It's the [primary database](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-2000-server/cc961761(v=technet.10)?redirectedfrom=MSDN) of the Active Directory and a copy of it is hosted in any domain controller (DC). It serves as a centralized repository for all the [domain’s objects](((65a7a0b2-a866-414f-b6af-04542a7e6656))) and their associated information.
- It's stored in `%systemroot%\NTDS\ntds.dit`, but during the Active Directory installation process the location can be customized.
	- Since the *NTDS.dit* is constantly used by AD processes such as the [*KDC*](((6565b3f7-61b6-4b2a-a59d-01d20e6acd96))), **it can't be copied** like any other file.
	- To export NTDS.dit's content, [various techniques](https://www.thehacker.recipes/ad/movement/credentials/dumping/ntds) can be employed:
	  id:: 65a94acc-e6fa-4bb0-89b3-c9944e0b3a53
		- NTDSUtil
		  logseq.order-list-type:: number
		- [[Shadow Copy]]
		  logseq.order-list-type:: number
		- NTFS structure parsing
		  logseq.order-list-type:: number
- #+BEGIN_PINNED
  The *NTDS.dit* acts as a **single source of truth** for the entire domain.
  #+END_PINNED
- Internally the *NTDS.dit* file is logically split into the following partitions:
	- **Schema Partition**: There is only one schema partition per forest and it is stored in all DCs of the forest.  It contains the definition of objects and rules for their manipulation and creation in an active directory.
	  logseq.order-list-type:: number
		- It is replicated to all DCs of the forest.
	- **Configuration Partition**: Just like schema partition, there is just one master configuration partition per forest and a second one on all DCs in a forest. It contains the forest-wide active directory topology including DCs and sites and services.
	  logseq.order-list-type:: number
		- It is replicated to all DCs in a forest.
	- **Domain Partition**: Many domain partitions exist per forest and they are stored on all DCs in a domain. They contain information about users, groups, computers and OUs.
	  logseq.order-list-type:: number
		- It is replicated to all DCs in a given domain.
	- **Application Partition**: This partition stores information about applications in an AD. Suppose 
	  logseq.order-list-type:: number
	  AD integrated DNS zones information is stored in this partition.
		- It is replicated only between designated DCs.
- Some confidential information stored in the *NTDS.dit* database (like user's password hashes) are encrypted with the PEK *(Password Encryption Key)* key.
  id:: 65a94684-cdb5-4d91-b092-4044b38ce47e
	- The PEK is the same across the whole domain, which means that it is the same on all the domain controllers.
	- **The PEK itself is also stored in the NTDS.dit** in an encrypted form. It is encrypted with the *BOOTKEY* which instead is different for each domain controller.
	- The BOOTKEY is stored in the DC's SYSTEM hive. Hence, in order to decrypt the juicy sensitive information in the *NTDS.dit*, we also need the SYSTEM hive of the DC from where we have extracted the *NTDS.dit* database.
	- The SYSTEM hive is located at `\system32\config\system` and can be extracted in the following ways
		- From the registry
		  logseq.order-list-type:: number
		  ```cmd
		  reg save HKLM\SYSTEM 'C:\Windows\Temp\system.save'
		  ```
		- Using the [same technique](((65a94acc-e6fa-4bb0-89b3-c9944e0b3a53))) we have used to export the *NITD.dit* file.
		  logseq.order-list-type:: number
- Decrypt the *NTDS.dit* file
  id:: 65a95382-e8f2-483f-8991-4b8bc67bcb7a
	- Once you've successfully exfiltrated the *NTDS.dit* and the *SYSTEM hive* from a DC, then you can proceed to crack them offline from your Kali machine and retrieve passwords' hashes for every domain-user.
		- With [[Impacket-secretsdump]]
		  id:: 65a956e6-ec7d-49d1-8c08-706e1ca7b8fe
		  ```bash
		  impacket-secretsdump -ntds ntds.dit.bak -system system.bak LOCAL
		  ```
			- `-ntds` to specify the NTDS.dit file
			- `-system` to specify the dump of the SYSTEM hive
			- `LOCAL` to parse the files locally
		- With [gosecretsdump](https://github.com/c-sto/gosecretsdump) (written in Go, so faster)
		  ```bash
		  gosecretsdump -ntds ntds.dit.save -system system.save
		  ```
			- `-ntds` to specify the NTDS.dit file
			- `-system` to specify the dump of the SYSTEM hive
- Since the *NTDS.dit* file is responsible for storing the entire directory, with users, groups, OUs, trusted domains etc. W can use the tool [ntdsdotsqlite](https://github.com/almandin/ntdsdotsqlite) to extract more information than just secrets.
	- ```bash
	  ntdsdotsqlite ntds.dit -o ntds.sqlite
	  ```
	- Of course, with the *SYSTEM hive* available it is able to extract credentials as well
	  ```bash
	  ntdsdotsqlite ntds.dit -o ntds.sqlite --system SYSTEM.hive
	  ```