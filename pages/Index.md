public:: true

- [whoami](https://gianlu.ca)
- Knowledge Base
	- Information Gathering
		- Passive
		- Active
			- Network
				- [Ports]([[Information Gathering Ports]])
			- Users Enumeration
				- [SMTP]([[User Enumeration via SMTP]])
	- Protocols
		- [[SMNP]]
		  background-color:: yellow
	- Attack Vectors
		- [[SQL Injection]]
			- [[Microsoft SQL]]
	- Initial Steps
		- [Windows]([[Windows Initial Steps]])
		- [Linux]([[Linux Initial Steps]])
		- [Webserver]([[Webserver Initial Steps]])
	- Exploit
		- [Windows]([[Windows Exploits]])
		- [Linux]([[Linux Exploits]])
	- [[Reverse Shells]]
		- [Windows]([[Windows Reverse Shells]])
		- [Linux]([[Linux Reverse Shells]])
	- Exfiltration
		- [Windows]([[Exfiltration in Windows]])
	- [[Active Directory]]
		- Enumeration
			- Manual
				- [Legacy Windows Tools]([[Manual AD Enumeration via Legacy Windows Tools]])
				- [PowerView]([[Manual AD Enumeration via PowerView]])
				- [[SMB]]
				  background-color:: yellow
			- Brute Force
				- [Users]([[Kerberos User Enumeration]])
		- [Privilege Escalation]([[AD Privilege Escalation]])
			- Windows Access Control Mechanisms
				- [Access Tokens]([[AD Access Tokens]])
				- [[Mandatory Integrity Control]]
				- [[User Account Control]]
			- [Situational Awareness]([[AD Situational Awareness]])
			- [Low-Hanging Fruit]([[Low-Hanging Fruit in Windows]])
				- [[PowerShell History]]
				  background-color:: pink
				- [[Password Spray]]
				  background-color:: yellow
				- [[Services Takeover]]
			- Exploit
				- The Potato Family
				- PrintSpoofer
				- The Ethernal Family
				- Zerologon
		- [Lateral Movement](https://attack.mitre.org/tactics/TA0008/)
			- [Tips]([[AD Lateral Movement Tips]])
			- [[WMI]]
			- [[WinRM]]
			  background-color:: yellow
			- [[PsExec]]
			- [[Pass-The-Hash]]
			  id:: 65a7a0af-5ba2-431e-8d4a-14234d7a95e7
			- [[Overpass-The-Hash]]
			- [[Pass-The-Ticket]]
			- [[DCOM]]
		- [Persistence]([[AD Persistence]])
			- [[DCSync]]
			- [[Golden Ticket]]
			- [[Shadow Copy]]
			  background-color:: pink
			- [[NTDS]]
		- Security Enforcement
			- [[Protected Users Security Group]]
		- Authentication
			- [[NTLM Authentication]]
			- [Kerberos]([[AD Kerberos Authentication]])
				- [[PKINIT]] (move this in [[Kerberos]])
				  background-color:: yellow
			- [[WDigest]]
			  background-color:: yellow
			- Supplement
				- [[RC4-HMAC]]
				  background-color:: yellow
		- [Attacks]([[Attack AD Authentication]])
			- [Password]([[AD Authentication Password Attack]])
			- [[Cached AD Credentials]]
			- [[AS-REQ Roasting]]
			- [[AS-REP Roasting]]
			- [[Kerberoasting]]
				- [Targeted Kerberoasting](logseq://graph/HTB-Notes?block-id=655e327e-5e4b-4260-828e-33941dad976c)
				  id:: 655e33c3-0c50-48d0-88d8-1c03abf25caa
			- [[Silver Tickets]]
			- [[Golden Ticket]]
			- [[DCSync]]
			- [[Abuse AD CS]]
			  background-color:: yellow
			- [[Shadow Copy]]
			- [[RC4 AS-REP Decryption]]
			  background-color:: yellow
			- Exploits
				- Zerologon
	- Awesome tools and where to find them
		- Network
			- [[Ligolo-ng]]
			- OpenSSH
				- Tunnelling
		- Windows
			- [[Mimikatz]]
			- [[CrackMapExec]]
			  background-color:: yellow
			- [[Kerbrute]]
			- [[Impacket]]
				- [GetNPUsers]([[Impacket GetNPUsers]])
				- [GetUserSPNs]([[Impacket_GetUserSPNs]])
				- [secretsdump]([[Impacket-secretsdump]])
				- [lookupsid]([[Impacket-lookupsid]])
			- [[Rubeus]]
			- [[Patator]]
			  background-color:: yellow
			- [Kerberoast]([[Kerberoast tool]])
			  background-color:: yellow
			- [[winPEAS]]
			  background-color:: yellow
			- [[Responder]]
			  background-color:: yellow
			- [[Evil-WinRM]]
			  background-color:: yellow
			- [[enum4linux]]
			- [[enum4linux-ng]]
			- Powercat
		- Cracking
			- [[Hashcat]]
			- [[John The Ripper]]
- Labs and CTFs
	- [[Hack The Box]]