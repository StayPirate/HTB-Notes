public:: true

- Kerberos User Enumeration is based on a feature of the protocol - the KDC service returns different errors for existing and non-existing users
	- `KDC_ERR_C_PRINCIPAL_UNKNOWN` - User doesn’t exist
	- `KDC_ERR_CLIENT_REVOKED` - User disabled/locked
	- `KDC_ERR_PREAUTH_REQUIRED` - User exists
- When the [AS_REQ](((655b1c01-e1e1-4be0-b245-ff8c9482df38))) is sent without [pre-authentication](((655b158a-a666-41e0-8076-e59942a7bb20))) it generates a Windows event ID [4768](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=4768) but it doesn’t count as a login failure.
  id:: 65671fc8-7211-4f80-88d5-27744b4ab82a
	- A lot of 4768 events for different users may point to a user enumeration attack attempt.
- [[Kerbrute]] can be used to automate the user enumeration process
  id:: 65672173-7151-439e-b4b4-ea71ff5f7128
	- id:: 6567218f-2f3a-4c8f-bdd1-ca629d69f2c1
	  ```bash
	  kerbrute_linux_amd64 userenum -d hpbank.local --dc dc01.hpbank.local usernames.txt
	  ```