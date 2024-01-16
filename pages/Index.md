public:: true

- #+BEGIN_QUOTE
  [whoami](https://gianlu.ca)
  #+END_QUOTE
- Knowledge Base
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
	- [[Active Directory]]
		- Enumeration
			- Manual
				- [Legacy Windows Tools]([[Manual AD Enumeration via Legacy Windows Tools]])
				- [PowerView]([[Manual AD Enumeration via PowerView]])
			- Brute Force
				- [Users]([[Kerberos User Enumeration]])
		- [Lateral Movement](https://attack.mitre.org/tactics/TA0008/)
			- [Tips]([[AD Lateral Movement Tips]])
			- [[WMI]]
			- [[WinRM]]
			- [[PsExec]]
			- [[Pass-The-Hash]]
			- [[Overpass-The-Hash]]
			- [[Pass-The-Ticket]]
			- [[DCOM]]
		- Persistence
			- Golden Ticket
			- Shadow Copies
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
			- [[DCSync]]
			- [[RC4 AS-REP Decryption]]
			  background-color:: yellow
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
			- [[Rubeus]]
			- [[Patator]]
			  background-color:: yellow
			- [Kerberoast]([[Kerberoast tool]])
			  background-color:: yellow
		- Cracking
			- [[Hashcat]]
			- [[John The Ripper]]
- Labs and CTFs
	- [[Hack The Box]]