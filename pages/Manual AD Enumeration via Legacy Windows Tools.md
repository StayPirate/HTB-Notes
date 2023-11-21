public:: true

- Enumerate domain users
  id:: 655b6438-8ebc-46f6-a345-059d33f3ad83
  ```cmd
  net user /domain
  ```
  ```cmd
  C:\Users\stephanie>net user /domain
  The request will be processed at a domain controller for domain corp.com.
  
  User accounts for \\DC1.corp.com
  
  -------------------------------------------------------------------------------
  Administrator            dave                     Guest
  iis_service              jeff                     jeffadmin
  jen                      krbtgt                   pete
  stephanie
  The command completed successfully.
  ```
	- Retrieve information about a specific user
	  ```cmd
	  net user jeffadmin /domain
	  ```
	  ```cmd
	  C:\Users\stephanie>net user jeffadmin /domain
	  The request will be processed at a domain controller for domain corp.com.
	  
	  User name                    jeffadmin
	  Full Name
	  Comment
	  User's comment
	  Country/region code          000 (System Default)
	  Account active               Yes
	  Account expires              Never
	  
	  Password last set            9/2/2022 4:26:48 PM
	  Password expires             Never
	  Password changeable          9/3/2022 4:26:48 PM
	  Password required            Yes
	  User may change password     Yes
	  
	  Workstations allowed         All
	  Logon script
	  User profile
	  Home directory
	  Last logon                   9/20/2022 1:36:09 AM
	  
	  Logon hours allowed          All
	  
	  Local Group Memberships      *Administrators
	  Global Group memberships     *Domain Users         *Domain Admins
	  The command completed successfully.
	  ```
	  *According to the output, jeffadmin is a part of the* ***Domain Admins*** *group.*
- Enumerate domain groups
  ```cmd
  net group /domain
  ```
  ```cmd
  C:\Users\stephanie>net group /domain
  The request will be processed at a domain controller for domain corp.com.
  
  Group Accounts for \\DC1.corp.com
  
  -------------------------------------------------------------------------------
  *Cloneable Domain Controllers
  *Debug
  *Development Department
  *DnsUpdateProxy
  *Domain Admins
  *Domain Computers
  *Domain Controllers
  *Domain Guests
  *Domain Users
  *Enterprise Admins
  *Enterprise Key Admins
  *Enterprise Read-only Domain Controllers
  *Group Policy Creator Owners
  *Key Admins
  *Management Department
  *Protected Users
  *Read-only Domain Controllers
  *Sales Department
  *Schema Admins
  The command completed successfully.
  ```
  *[Here](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-groups#default-active-directory-security-groups) a list of all the default AD groups.*
	- Retrieve the members list for a specific group
	  ```cmd
	  net group "Sales Department" /domain
	  ```
	  ```cmd
	  PS C:\Tools> net group "Sales Department" /domain
	  The request will be processed at a domain controller for domain corp.com.
	  
	  Group name     Sales Department
	  Comment
	  
	  Members
	  
	  -------------------------------------------------------------------------------
	  pete                     stephanie
	  The command completed successfully.
	  ```
	  *This reveals that pete and stephanie are members of the *Sales Department* group.*
	- id:: 6559cba1-6866-4f6c-8e30-d9bfd9a7df0f
	  #+BEGIN_CAUTION
	  The `net.exe` tool **only lists *user* objects**, not group objects. That means that all users part of the *Sales Department* via *nested groups* are not shown!
	  #+END_CAUTION
	- id:: 6559cd1a-af38-44ff-b7ba-396d3c250b67
	  #+BEGIN_NOTE
	  When a group is part of other group it is called ***Nested group***.
	  
	  E.g. If Joe is part of the *Development Department* group which in turn is part of the *Sales Department* group, then Joe is also part of the *Sales Department* group by inheritance.
	  #+END_NOTE
- Enumerate SPNs
  id:: 6559cc20-1bf4-4d59-a498-f183b3ae7fc6
	- For a definition of SPN check [here](((6559f91b-875c-4d67-b318-ebc7478a9d7a))).
	- By looking at the previously retrieved domain user list we spotted `iis_service`. Let's check it out:
	  ```cmd
	  c:\Tools>setspn -L iis_service
	  Registered ServicePrincipalNames for CN=iis_service,CN=Users,DC=corp,DC=com:
	          HTTP/web04.corp.com
	          HTTP/web04
	          HTTP/web04.corp.com:80
	  ```