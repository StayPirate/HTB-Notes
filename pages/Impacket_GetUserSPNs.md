public:: true
alias:: GetUserSPNs

- [Source code](https://github.com/fortra/impacket/blob/master/examples/GetUserSPNs.py)
- Deafult location in Kali: `/usr/bin/impacket-GetUserSPNs`
- This script is part of [[Impacket]] and can be used to find [Service Principal Names](((655e0fad-5b48-42ce-b82a-09cd0e4a9322))) that are associated with AD user account.
- *GetUserSPNs* implements a standalone Kerberos protocol which is used through a device connected on a network. Because of that it requires credentials for a domain account to perform the roasting, since a TGT needs to be requested for use in the later service ticket requests.
- Retrieve the list of [SPN](((655e0fad-5b48-42ce-b82a-09cd0e4a9322)))s associated to AD user accounts
  {{embed ((655e2491-e204-4c69-859d-53d7425c4419))}}
- Retrieve [TGS-REP](((655b6438-ee0f-4168-8a40-754613d2b793)))s for Kerberoastable users
  {{embed ((655e2ce7-cc6b-42f9-8adf-a6e93a8956dd))}}
- #+BEGIN_WARNING
  If you get the error `KRB_AP_ERR_SKEW(Clock skew too great)`, then you need to synchronize the time of the Kali machine with the domain controller.
  #+END_WARNING