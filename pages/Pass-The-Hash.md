public:: true

- The Pass the Hash (PtH) technique allows an attacker to authenticate to a remote system or service using a user's NTLM hash instead of the associated plaintext password. Note that this will not work for [[AD Kerberos Authentication]] but only for servers or services using [[NTLM Authentication]].
- This lateral movement sub-technique is also mapped in the MITRE Framework under the [Use Alternate Authentication Material](https://attack.mitre.org/techniques/T1550/002/) general technique.