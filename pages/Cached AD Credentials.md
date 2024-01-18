public:: true

- {{embed ((65686104-d39c-4820-9752-9963fae90657))}}
- Since Microsoft's implementation of Kerberos makes use of *single sign-on*, password hashes must be stored somewhere in order to allow TGT renewal request.
  id:: 6564d528-7b7b-4d33-b49c-36fe2d615343
	- In modern versions of Windows, these hashes are stored in the [*Local Security Authority Subsystem Service*](((65663778-526d-48d4-a36d-3177ea335c2a))) ([LSASS](https://en.wikipedia.org/wiki/Local_Security_Authority_Subsystem_Service)) memory space.
	- the *LSASS* process is part of the operating system and runs as SYSTEM. If we manage to get access to SYSTEM (or local administrator) permissions we can then dump the process' memory to retrieve the hashes stored on a target machine. The most complete tool for this purpose is probably [[Mimikatz]].
- Active Directory Certificate Services
	- Microsoft provides the AD role *Active Directory Certificate Services* ([ADCS](https://docs.microsoft.com/en-us/training/modules/implement-manage-active-directory-certificate-services/)) to implement a PKI, which exchanges digital certificates between authenticated users and trusted resources.
	- Certificates may be marked as having a [*non-exportable* private key](https://techcommunity.microsoft.com/t5/core-infrastructure-and-security/marking-private-keys-as-non-exportable-with-certutil-importpfx/ba-p/1128390) for security reasons. If so, a private key associated with a certificate cannot be exported even with administrative privileges. Again Mimikatz [can help to export](((655b7624-0ef9-481b-96a1-2aa762c3f0dd))) certificates' private keys regardless how the *exportable* option is set.
	  id:: 655b76db-6fdf-470e-9bea-1d662dcfd6c1