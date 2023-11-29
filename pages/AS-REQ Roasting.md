public:: true
alias:: ASREQRoasting

- If an attacker is able to intercept a first [AS-REQ](((655b1c01-e1e1-4be0-b245-ff8c9482df38))) request which contains [pre-authentication information](((655b158a-a666-41e0-8076-e59942a7bb20))) (in most cases) they can try to run an offline bruteforce attack to retrieve the user's password.
- That's a different attack than [[AS-REP Roasting]] because it doesn't require the target user to have pre-authentication disabled and trick the KDC to return data encrypted with the targeted user's password. Indeed you need to somehow sniff a legit first AS-REQ from the user.
- Since that first AS-REQ contains [`PA-ENC-TIMESTAMP`](((655b158a-a666-41e0-8076-e59942a7bb20))) structure encrypted with the user's password we can try to crack this field offline.
- [[Hashcat]] provides the following modes to crack an AS-REQ
  id:: 6567302f-bb7b-413d-bf84-4ac00b10e784
	- ```
	  7500 - Kerberos 5, etype 23 (RC4), Pre-Auth (3599.6 MH/s RTX4090)
	  19800 - Kerberos, etype 17 (AES128), Pre-Auth (5135.3 kH/s RTX4090)
	  19900 - Kerberos, etype 18 (AES256), Pre-Auth (2571.6 kH/s RTX4090)
	  ```
	- Crack **Kerberos AS-REQ** hashes via [[Hashcat]] with the [`-m 19900`](https://hashcat.net/wiki/doku.php?id=example_hashes) option.
	  id:: 656730da-83ef-4f36-90b4-55a504e21369
	  ```bash
	  # $krb5pa$18$da$HPBANK.LOCAL$b4cdf79f8ab3667a19247c601b62630b6e9393e48f67c044bdc3f98dab67357e4803346c6856e67b8e44e169cc71dd636ca363a853885d7b
	  sudo hashcat -m 19900 hashes.asreqroast /usr/share/wordlists/rockyou.txt \
	  	-r /usr/share/hashcat/rules/best64.rule --force
	  ```
- #+BEGIN_WARNING
  TBH I do not understand how hashcat (or similar tools) knows if the decryption successfully happened, and understand when to exit a successfully the brute-force attempt.
  The `PA-ENC-TIMESTAMP` structure
  ```
  PA-ENC-TIMESTAMP        ::= EncryptedData -- PA-ENC-TS-ENC
  PA-ENC-TS-ENC           ::= SEQUENCE {
           patimestamp     [0] KerberosTime -- client's time --,
           pausec          [1] Microseconds OPTIONAL
   }
  ```
  The KerberosTime value used for the timestamp is an ASCII string of the form `YYYYMMDDHHmmssZ`.
  The Microseconds is an unsigned integer, any number from 0 to 999999.
  - The only thing I can think of is that after any attempt it checks if `patimestamp` matches the form of `YYYYMMDDHHmmssZ`.
  #+END_WARNING