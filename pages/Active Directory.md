public:: true

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