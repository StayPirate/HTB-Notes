public:: true

- The **[Domain Controller](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc786438(v=ws.10))** is a server that provides all the services needed for managing activities occurring in your IT environment. In particular, it makes sure each person is who they claim to be (*authentication*) and allow them to access only the data they’re allowed to use (*authorization*). There are several services running on it, like:
  id:: 65a7a0b2-f9d2-436b-b4f2-369427fa7489
	- The **AD** *([Active Directory](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview))*, it is a database (directory) holding information about AD objects in the domain. Common types of AD objects include: User, Groups, Computers, Applications, Printers, Shared folders.
	  id:: 65a7a0b2-a866-414f-b6af-04542a7e6656
	- The **KDC** *([Key Distribution Center](https://learn.microsoft.com/en-us/windows/win32/secauthn/key-distribution-center))* holds a database of the keys used in the authentication process and consists of two main parts:
	  id:: 6565b3f7-61b6-4b2a-a59d-01d20e6acd96
		- The  *Authentication Service* (AS) responsible for authenticating clients and issuing [TGT](((655b1bc6-5c5d-4c70-9d2b-f3f3d6458cb9)))s.
		  logseq.order-list-type:: number
		- The *Ticket Granting Service* (TGS) issues [tickets](((655a24c7-b91e-4a45-8468-c565395f566e))) for connection to computers in its own domain.
		  logseq.order-list-type:: number
	- According to [Microsoft](https://learn.microsoft.com/en-us/windows/win32/secauthn/key-distribution-center), KDC and AD are located on the domain controller. Both services are started automatically by the domain controller's *Local Security Authority* (LSA) and run as part of the LSA's process.
	  id:: 65663778-526d-48d4-a36d-3177ea335c2a
		- collapsed:: true
		  #+BEGIN_CENTER
		  ![image.png](../assets/image_1701166201207_0.png)
		  #+END_CENTER
			- {{renderer excalidraw, excalidraw-2023-11-28-10-52-09}}
- Principal Names
	- In [[Kerberos]], a principal name is a unique identifier for a network entity, such as a *user* or *service*, that participates in Kerberos authentication and authorization.
	- Service Principal Name ([SPN](https://learn.microsoft.com/en-us/windows/win32/ad/service-principal-names) or *sname*)
	  id:: 655e0fad-5b48-42ce-b82a-09cd0e4a9322
		- Identifies a particular service offered by a particular host within a domain and it's assigned to application instances when they are registered in the Active Directory *(Kerberized services)*.
		- The *SPN identity* is a domain user account that has been mapped to the SPN. This mapping allows the KDC to identify which Kerberos keys should be used in the communication.
			- If a SPN is associated to a **domain user account**, then the [[Kerberoasting]] attack can be execute attempting to retrieve the user's password.
			- For computer accounts, [managed service accounts](https://techcommunity.microsoft.com/t5/ask-the-directory-services-team/managed-service-accounts-understanding-implementing-best/ba-p/397009), and [group-managed service accounts](https://learn.microsoft.com/en-us/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview) the password is randomly generated, complex, and 120 characters long, **making cracking infeasible**.
			  id:: 655e3185-f921-4b07-bb00-e397a2486fc6
		- The SPN consists of a string on the form of: `service class`/`fqdn`@`REALM`.
			- The `service class` can loosely be thought of as the protocol for the service.
				- The list of service classes that are built-in to Windows are [listed here](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc772815(v=ws.10)?redirectedfrom=MSDN#service-principal-names).
			- The `fqdn` is the *[fully qualified domain name](https://en.wikipedia.org/wiki/Fully_qualified_domain_name)* of the computer hosting the service.
			- `REALM` is a Kerberos term to refer what in AD-world is called *domain*
			- Examples:
				- IMAP/mail.example.com@EXAMPLE.COM
				- HTTP/web.hpbank.local
				- MSSQLSvc/MSSQL01.hpbank.local
				- CIFS/DC01.hpbank.local
	- User Principal Name (UPN or *cname*)
		- Similar to the SPN but for user objects
- Security Principals
  id:: 65ae443c-68a0-4b2e-b534-b7cf8934ec14
	- A Windows security principal is an authenticated entity, like a **user**, **computer**, or **group of users** or **computers**. Each security principal has a unique identifier that's known as a SID.
	- Security Identifiers (SID)
	  id:: 65ae6139-ddd3-44a1-906e-f0302a9151e7
		- A security identifier is used to uniquely identify a *security principal* or *security group*.
		- The system authority (*[Local Security Authority](https://learn.microsoft.com/en-us/windows-server/security/credentials-protection-and-management/configuring-additional-lsa-protection)* for local entities, or *[DC](((65a7a0b2-f9d2-436b-b4f2-369427fa7489)))* for domain ones), generates the SID that identifies a particular account or group at the time the account or group is created. When a SID has been used as the unique identifier for a user or group, it can never be used again to identify another user or group.
		- SID Architecture
			- A security identifier is a binary data structure that contains a variable number of values in the form of `S-R-X-Y1-Y2-Yn-1-Yn`.
				- `S` is a literal which indicates that the string is a SID.
				- `R` is the revision number and is **always set to 1**.
				- `X` identifies the highest level of authority that can issue SIDs for a particular type of security principal.
					- `1` is the *World Authority*.
					- `5` specifies the *NT Authority* and is used for local or domain users and groups. This is the most common authority value.
				- `Y1-Y2-Yn-1-Yn` represents the most important information in a SID, which is contained in a series of one or more **subauthority** values.
					- `Y1-Y2-Yn-1` is the the **domain identifier** and can identify a domain in an enterprise.
						- `domainidentifier` is the SID of the domain for domain users.
						- `rootdomainidentifier` is the SID of the forest root domain.
						- `localmachine` is the SID of the local machine for local users.
						- `32` for built-in principals.
					- `Yn` is the **Relative Identifier** (RID) and identifies a particular account or group relative to a domain.
			- SID Examples
				- The SID for a **local user** on a Windows system is
				  `S-1-5-21-1336799502-1441772794-948155058-1001`
					- `21-1336799502-1441772794-948155058` are the sub-authorities of the identifier authority.
					- `1001` is the user RID
						- Because the RID starts at 1000 for nearly all principals, this implies that this is the second local user created on the system.
				- The SID of the [built-in Administrator group](((65ae5e3a-c0ae-412c-8fe8-932a85560b17))) is `S-1-5-32-544`
					- `1` is the revision level
					- `5` is the identifier authority value (NT Authority)
					- `32` is the domain identifier (built-in principals)
					- `544` is the relative identifier (RID) for the Administrator group
				- The SID for the global [Domain Admins](((65ae5d70-a04c-448e-98c0-5bfe849f1172))) group for a specific domain is
				  `S-1-5-21-1004336348-1177238915-682003330-512`
					- `1` is the revision level
					- `5` is the identifier authority value (NT Authority)
					- `21-1004336348-1177238915-682003330` is the domain identifier
					- `512` is the relative identifier (RID) for the Domain Admins group
		- [Well](https://learn.microsoft.com/en-us/windows/win32/secauthz/well-known-sids)-[known](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-identifiers) Security Identifiers
			- Well-known SIDs are default built-in entities in Windows used to identify generic groups and users. These SIDs have values that remain constant across all operating systems, and the RID is lower than 1000.
			- #+BEGIN_NOTE
			  Built-in *Well-known Security Principals* **cannot be renamed** or **deleted**.
			  #+END_NOTE
				- **Nobody** (group): `S-1-0-0`
					- *AKA Null SID, a group with no members. This is often used when a SID value is not known.*
				- **Everyone** (group): `S-1-1-0`
					- *AKA World, a group that includes all users.*
				- **Local**: `S-1-2-0`
					- *Users who log on to terminals locally (physically) connected to the system.*
				- **Authenticated Users** (group): `S-1-5-11`
					- *A group that includes all users and computers with identities that have been authenticated.*
					- *This group includes authenticated security principals from any trusted domain, not only the current domain.*
					- *Authenticated Users doesn't include Guest even if the Guest account has a password.*
				- **System**:  `S-1-5-18`
					- AKA LocalSystem.
					- *An identity that's used locally by the operating system and by services that are configured to sign in as LocalSystem.*
					- *System is a hidden member of Administrators. That is, any process running as System has the SID for the built-in Administrators group in its access token.*
				- **Administrator**: `S-1-5-domainidentifier-500`
					- *A user account for the system administrator. Every computer has a local Administrator account and every domain has a domain Administrator account.*
					- *The Administrator account is the first account created during operating system installation.*
					- *The account can't be deleted, disabled, or locked out, but it can be renamed.*
					- *By default, the Administrator account is a member of the Administrators group, and it can't be removed from that group.*
				- **Administrators** (group): `S-1-5-32-544`
				  id:: 65ae5e3a-c0ae-412c-8fe8-932a85560b17
					- *Administrators of the domain.*
				- **KRBTGT**: `S-1-5-domainidentifier-502`
					- *A user account that's used by the [Key Distribution Center](((6565b3f7-61b6-4b2a-a59d-01d20e6acd96))) (KDC) service. The account exists only on domain controllers.*
				- **Domain Admins** (group): `S-1-5-domainidentifier-512`
				  id:: 65ae5d70-a04c-448e-98c0-5bfe849f1172
					- *A global group with members that are authorized to administer the domain.*
					- *By default, the Domain Admins group is a member of the Administrators group on all computers that have joined the domain, including domain controllers.*
				- **Enterprise Admins** (group): `S-1-5-rootdomainidentifier-519`
					- *A group that exists only in the forest root domain.*
					- *By default, the only member of Enterprise Admins is the Administrator account for the forest root domain.*
					- *Members of this group have full access to all domains in the Active Directory forest.*
				- **Domain Users** (group): `S-1-5-domainidentifier-513`
					- *A global group that includes all users in a domain.*
				- **Domain Computers** (group): `S-1-5-domainidentifier-515`
					- *A global group that includes all computers that have joined the domain, excluding domain controllers.*
				- **Domain Controllers** (group): `S-1-5-domainidentifier-516`
					- *A global group that includes all domain controllers in the domain.*
				- **Protected Users** (group): `S-1-5-domainidentifier-525`
					- *A global group that is afforded additional protections against authentication security threats.*
				- **Self**: `S-1-5-10`
					- *A placeholder in an ACE for a user, group, or computer object in Active Directory. When you grant permissions to Self, you grant them to the security principal that's represented by the object. During an access check, the operating system replaces the SID for Self with the SID for the security principal that's represented by the object.*
				- **Creator Owner**: `S-1-3-0`
					- *A security identifier to be replaced by the security identifier of the user who created a new object. This SID is used in inheritable ACEs.*
				- **Creator Group**: `S-1-3-1`
					- *A security identifier to be replaced by the primary-group SID of the user who created a new object. Use this SID in inheritable ACEs.*
				- **All Services** (group): `S-1-5-80-0`
					- *A group that includes all service processes configured on the system. Membership is controlled by the operating system.*
		- {{embed ((6568a492-a226-4069-a5ef-127a23fb9de5))}}
- Computer accounts
	- Password *(the following information are the default ones, then each environment can override them)*
	  id:: 6568a1fb-58d3-4402-bbb5-8c3d1bca729e
		- Auto-generated
		- Complexity: High
		- Lenght: 120
		- Rotation period: 30 days
- Find the Primary Domain Controller
	- Within a domain there might be multiple DCs. In  that case the *domain name* could potentially resolve to the IP address of any of them.
	  
	  To make our enumeration as accurate as possible, we should look for the DC that
	  holds the most updated information. This is known as the [*Primary Domain Controller* (PDC)](https://portal.offsec.com/courses/pen-200/books-and-videos/modal/modules/active-directory-introduction-and-enumeration/active-directory-manual-enumeration/enumerating-active-directory-using-powershell-and-net-classes#fn6). There can be only one PDC in a domain. To find the PDC, we need to find the DC holding the ***PdcRoleOwner*** property.
	  ```powershell
	  PS C:\> [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()
	  ```
	  ```powershell
	  PS C:\> [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()
	  
	  Forest                  : corp.com
	  DomainControllers       : {DC1.corp.com}
	  Children                : {}
	  DomainMode              : Unknown
	  DomainModeLevel         : 7
	  Parent                  :
	  PdcRoleOwner			: DC1.corp.com
	  RidRoleOwner            : DC1.corp.com
	  InfrastructureRoleOwner : DC1.corp.com
	  Name                  	: corp.com
	  ```
	  Only return the PDC:
	  ```powershell
	  PS C:\> [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain().PdcRoleOwner.Name
	  DC1.corp.com
	  ```