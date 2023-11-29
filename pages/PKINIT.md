- #+BEGIN_QUOTE
  [[MS-PKCA](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-pkca/d0cf1763-3541-4008-a75f-a577fa5e8c5b)]: Public Key Cryptography for Initial Authentication (PKINIT) in Kerberos Protocol *(Microsoft specification)*
  [[RFC4556](https://datatracker.ietf.org/doc/html/rfc4556)]: Public Key Cryptography for Initial Authentication in Kerberos (PKINIT) *(Kerberos specification)*
  #+END_QUOTE
- The Public Key Cryptography for Initial Authentication in Kerberos (PKINIT) protocol enables the use of public key cryptography in the initial authentication exchange (that is, in the [Authentication](((6564d528-ff73-410b-a538-c3dbcd9d039e))) [Service](((6564d528-4967-4c86-82aa-9168c47ab29c))) (AS) exchange) of the [[Kerberos]] protocol.
- The KDC should have an X.509 public key certificate, issued by a certificate authority (CA) and trusted by the clients.
- TODO define the workflow
  background-color:: pink
- TODO draw exchanged packages for this workflow
  background-color:: pink
- See the great [work](https://www.chudamax.com/posts/kerberos-102-overview/#pkinit) done by Maksim for inspiration