public:: true

- [Source code](https://github.com/fortra/impacket/blob/impacket_0_10_0/examples/GetNPUsers.py)
- Deafult location in Kali: `/usr/bin/impacket-GetNPUsers`
- This script is part of [[Impacket]] and attempts to list and get TGTs for those users that have the property [*Do not require Kerberos preauthentication*](((655b158a-a666-41e0-8076-e59942a7bb20))) set and automatically generates [[Hashcat]]/[[JTR]] crackable strings.
- Find users with Kerberos Pre-Authentication disabled
  {{embed ((5971e61f-2ed6-4dd2-91a6-1752dfb51de7))}}
- Retrieve [[ASREPRoast]] hashes
  id:: 655ce01b-c0e5-4549-bbd9-9ec9747dc05a
  {{embed ((655cf20b-a6e7-4003-8049-8d922905d0b1))}}