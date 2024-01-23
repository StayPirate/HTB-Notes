public:: true

- After having gotten access as unprivileged user in a Windows machine, we should obtain as much as information as possible to understand the nature of the compromised machine and discover possible vectors for privilege escalation.
- The key pieces of information we should always obtain are:
	- Username and hostname
	- Group memberships of the current user
	- Existing users and groups
	- Operating system, version and architecture
	- Network information
	- Installed applications
	- Running processes
- Username and hostname
	- We can simply run `whoami` to get these information.
	  ```cmd
	  C:\Users\dave>whoami
	  whoami
	  clientwk220\dave
	  ```
		- Now we know that we have command execution as user *dave*
		- The hostname of the system is *clientwk220*
			- It seems to be a workstation rather than a server.
- Group memberships
	- We can run `whoami /groups`
	  ```cmd
	  C:\Users\dave> whoami /groups
	  whoami /groups
	  
	  GROUP INFORMATION
	  -----------------
	  
	  Group Name                             Type             SID                                            Attributes                                        
	  ====================================== ================ ============================================== ==================================================
	  Everyone                             Well-known group S-1-1-0                                        Mandatory group, Enabled by default, Enabled group
	  CLIENTWK220\helpdesk                 Alias            S-1-5-21-2309961351-4093026482-2223492918-1008 Mandatory group, Enabled by default, Enabled group
	  BUILTIN\Remote Desktop Users         Alias            S-1-5-32-555                                   Mandatory group, Enabled by default, Enabled group
	  BUILTIN\Users                        Alias            S-1-5-32-545                                   Mandatory group, Enabled by default, Enabled group
	  NT AUTHORITY\BATCH                   Well-known group S-1-5-3                                        Mandatory group, Enabled by default, Enabled group
	  CONSOLE LOGON                        Well-known group S-1-2-1                                        Mandatory group, Enabled by default, Enabled group
	  NT AUTHORITY\Authenticated Users     Well-known group S-1-5-11                                       Mandatory group, Enabled by default, Enabled group
	  ... 
	  ```
		- *dave* is a member of the **helpdesk group** and helpdesk staff often have additional permissions and access compared to standard users.
		- *dave* is a member of **BUILTIN\Remote Desktop Users** and the membership of this group offers the possibility to **connect to the system via RDP**.
	- We can also use  `net user` for domain users.
	  ```cmd
	  C:\Users\dave> net user steve
	  net user steve
	  User name                    steve
	  ...
	  Last logon                   6/16/2022 1:03:52 PM
	  
	  Logon hours allowed          All
	  
	  Local Group Memberships      *helpdesk             *Remote Desktop Users 
	                               *Remote Management Use*Users                
	  ...
	  ```
- Existing users and groups
	- To enumerate local users we can use the [`net user`](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc771865(v=ws.11)) command or the [`Get-LocalUser`](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.localaccounts/get-localuser?view=powershell-5.1) cmdlet in PowerShell.
	  ```powershell
	  PS C:\Users\dave> Get-LocalUser
	  Get-LocalUser
	  
	  Name               Enabled Description                                                                              
	  ----               ------- -----------                                                                              
	  Administrator      False   Built-in account for administering the computer/domain
	  BackupAdmin        True
	  dave               True    dave 
	  daveadmin          True 
	  DefaultAccount     False   A user account managed by the system.
	  Guest              False   Built-in account for guest access to the computer/domain
	  steve              True
	  ... 
	  ```
		- The built-in Administrator account **is disabled**.
		- Two presumed regular users called *steve* and *dave* exist.
		- Two other users named *daveadmin* and *BackupAdmin* are present and both contain the word **admin**.
	- To enumerate local groups we can use the [`net localgroup`](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc725622(v=ws.11)) command or the [`Get-LocalGroup`](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.localaccounts/get-localgroup?view=powershell-5.1) cmdlet in PowerShell.
	  ```powershell
	  PS C:\Users\dave> Get-LocalGroup
	  Get-LocalGroup
	  
	  Name                                Description                                                                      
	  ----                                -----------                                                                     
	  adminteam                  Members of this group are admins to all workstations on the second floor
	  BackupUsers 
	  helpdesk
	  ...
	  Administrators                      Administrators have complete and unrestricted access to the computer/domain
	  ...
	  Remote Desktop Users                Members in this group are granted the right to logon remotely
	  ...  
	  ```
		- Three non-standard groups *adminteam*, *BackupUsers*, and *helpdesk* exist.
			- According to its description members of the *adminteam* group are administrators to workstations on the second floor.
		- Standard, but interesting groups
			- Backup Operators
				- Members of *Backup Operators* can **backup and restore all files** on a computer, even **those files they don't have permissions** for.
				- We must not confuse this group with non-standard groups such as *BackupUsers* in the above example.
			- Remote Desktop Users
				- Members of *Remote Desktop Users* can access the system with RDP.
			- Remote Management Users
				- Members of *Remote Management Users* can access the system with [[WinRM]].
	- To enumerate members of a local group we can use the [`Get-LocalGroupMember`](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.localaccounts/get-localgroupmember?view=powershell-5.1) cmdlet in PowerShell.
	  ```powershell
	  PS C:\Users\dave> Get-LocalGroupMember adminteam
	  Get-LocalGroupMember adminteam
	  
	  ObjectClass Name                PrincipalSource
	  ----------- ----                ---------------
	  User        CLIENTWK220\daveadmin Local
	  ```
	  ```powershell
	  PS C:\Users\dave> Get-LocalGroupMember Administrators
	  Get-LocalGroupMember Administrators
	  
	  ObjectClass Name                      PrincipalSource
	  ----------- ----                      ---------------
	  User        CLIENTWK220\Administrator Local          
	  User        CLIENTWK220\daveadmin     Local
	  User        CLIENTWK220\backupadmin   Local  
	  ```
		- *daveadmin* is a member of *adminteam*. However, *adminteam* is not listed in the *local Administrators group*.
		- Apart from the disabled local *Administrator* account, the users *daveadmin* and *BackupAdmin* are members of the *local Administrators group*.
- Operating system, version and architecture
	- We can run `systeminfo` to gather this information.
	  ```cmd
	  C:\Users\dave> systeminfo
	  systeminfo
	  
	  Host Name:                 CLIENT220
	  OS Name:                   Microsoft Windows 11 Pro
	  OS Version:                10.0.22000 N/A Build 22000
	  ...
	  System Type:               x64-based PC
	  ...
	  ```
		- The OS is **Windows 11 Pro**.
		- From the build number **we know** which exact version is. In this case is **version 21H2**.
		- The architecture is 64 bit.
- Network information
	- Network information may help us to identify **new services** or even **access to other networks**.
	- Enumerate network interfaces
		- We can use the [`ipconfig`](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/ipconfig) command.
		  ```cmd
		  C:\Users\dave> ipconfig /all
		  ipconfig /all
		  
		  Windows IP Configuration
		  
		     Host Name . . . . . . . . . . . . : clientwk220
		     Primary Dns Suffix  . . . . . . . : 
		     Node Type . . . . . . . . . . . . : Hybrid
		     IP Routing Enabled. . . . . . . . : No
		     WINS Proxy Enabled. . . . . . . . : No
		  
		  Ethernet adapter Ethernet0:
		  
		     Connection-specific DNS Suffix  . : 
		     Description . . . . . . . . . . . : vmxnet3 Ethernet Adapter
		     Physical Address. . . . . . . . . : 00-50-56-8A-80-16
		     DHCP Enabled. . . . . . . . . . . : No
		     Autoconfiguration Enabled . . . . : Yes
		     Link-local IPv6 Address . . . . . : fe80::cc7a:964e:1f98:babb%6(Preferred) 
		     IPv4 Address. . . . . . . . . . . : 192.168.50.220(Preferred) 
		     Subnet Mask . . . . . . . . . . . : 255.255.255.0
		     Default Gateway . . . . . . . . . : 192.168.50.254
		     DHCPv6 IAID . . . . . . . . . . . : 234901590
		     DHCPv6 Client DUID. . . . . . . . : 00-01-00-01-2A-3B-F7-25-00-50-56-8A-80-16
		     DNS Servers . . . . . . . . . . . : 8.8.8.8
		     NetBIOS over Tcpip. . . . . . . . : Enabled
		  ```
			- The system is not configured to get an IP address via DHCP, but it was set manually.
			- With a *MAC address* [lookup](https://macvendors.com/) we can confirm that it is a VMware virtual machine.
			- The *DNS server*, *gateway*, and *subnet mask* are other interesting pieces of information.
	- Enumerate routes
		- We can use the [`route`](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/route_ws2008) command.
		  ```cmd
		  C:\Users\dave> route print
		  route print
		  ===========================================================================
		  Interface List
		    6...00 50 56 8a 80 16 ......vmxnet3 Ethernet Adapter
		    1...........................Software Loopback Interface 1
		  ===========================================================================
		  
		  IPv4 Route Table
		  ===========================================================================
		  Active Routes:
		  Network Destination        Netmask          Gateway       Interface  Metric
		            0.0.0.0          0.0.0.0   192.168.50.254   192.168.50.220    271
		          127.0.0.0        255.0.0.0         On-link         127.0.0.1    331
		          127.0.0.1  255.255.255.255         On-link         127.0.0.1    331
		    127.255.255.255  255.255.255.255         On-link         127.0.0.1    331
		       192.168.50.0    255.255.255.0         On-link    192.168.50.220    271
		     192.168.50.220  255.255.255.255         On-link    192.168.50.220    271
		     192.168.50.255  255.255.255.255         On-link    192.168.50.220    271
		          224.0.0.0        240.0.0.0         On-link         127.0.0.1    331
		          224.0.0.0        240.0.0.0         On-link    192.168.50.220    271
		    255.255.255.255  255.255.255.255         On-link         127.0.0.1    331
		    255.255.255.255  255.255.255.255         On-link    192.168.50.220    271
		  ===========================================================================
		  Persistent Routes:
		    Network Address          Netmask  Gateway Address  Metric
		            0.0.0.0          0.0.0.0   192.168.50.254  Default 
		  ===========================================================================
		  ...
		  ```
			- No routes for any previously unknown networks are shown.
	- Enumerate active network connections
		- We can use the [`netstat`](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/netstat) command.
		  ```cmd
		  C:\Users\dave> netstat -ano
		  netstat -ano
		  
		  Active Connections
		  
		    Proto  Local Address          Foreign Address        State           PID
		    TCP    0.0.0.0:80             0.0.0.0:0              LISTENING       6824
		    TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       960
		    TCP    0.0.0.0:443            0.0.0.0:0              LISTENING       6824
		    TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
		    TCP    0.0.0.0:3306           0.0.0.0:0              LISTENING       1752
		    TCP    0.0.0.0:3389           0.0.0.0:0              LISTENING       1084
		    TCP    0.0.0.0:5040           0.0.0.0:0              LISTENING       3288
		  ...
		    TCP    192.168.50.220:139     0.0.0.0:0              LISTENING       4
		    TCP    192.168.50.220:3389    192.168.119.4:33060    ESTABLISHED     1084
		    TCP    192.168.50.220:4444    192.168.119.3:51082    ESTABLISHED     2044
		  ...
		  ```
			- `-a` to display all active TCP connections as well as TCP and UDP ports.
			- `-n` to disable name resolution.
			- `-o` to show the process ID for each connection.
			- The ports `80` and `443` are listening, usually indicating that a web server is running on the system.
			- The listening port `3306` is indicative of a running MySQL server.
			- The listening port `3389` is indicative that the system can be accessed via RDP (you need the user password [or]([[Pass-The-Hash]]) its [hash](https://www.kali.org/blog/passing-hash-remote-desktop/) to log via RDP).
			- Port `4444` is the bind shell we are using to access the system, indeed `192.168.119.3` is our IP address.
				- It doesn't shown as a listening port because the `netcat` command we used to run the bind shell does not fork after the first successful connection.
			- The established connection to port `3389` from the IP `192.168.119.4` indicates that we are not the only user connected to the system at that moment.
				- That also a good sign, because if we'll manage to elevate our privileges we could attempt to extract user's credentials with [[Mimikatz]].
- Installed applications
	- We can use the [`Get-ItemProperty`](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-itemproperty?view=powershell-7.2) cmdlet in PowerShell to query the *Windows Registry* to list all the installed 32 and 64 bit installed applications.
		- 32 bit
		  id:: 65afa6b7-48a0-4e1e-bed6-26b11224ff1e
		  ```powershell
		  PS C:\Users\dave> Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname 
		  Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname
		  
		  displayname                                                       
		  -----------                                                       
		  KeePass Password Safe 2.51.1                                      
		  Microsoft Edge                                                    
		  Microsoft Edge Update                                             
		  Microsoft Edge WebView2 Runtime                                   
		  ...
		  Microsoft Visual C++ 2015-2019 Redistributable (x86) - 14.28.29913
		  Microsoft Visual C++ 2019 X86 Additional Runtime - 14.28.29913    
		  Microsoft Visual C++ 2019 X86 Minimum Runtime - 14.28.29913       
		  Microsoft Visual C++ 2015-2019 Redistributable (x64) - 14.28.29913
		  ```
			- *KeePass* might indicate that keyrings *(\*.kdbx)* are present in the system.
		- 64 bit
		  ```powershell
		  PS C:\Users\dave> Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname
		  Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname
		  DisplayName                                                   
		  -----------                                                   
		  7-Zip 21.07 (x64)                                             
		  ...
		  XAMPP
		  VMware Tools                                                  
		  Microsoft Visual C++ 2019 X64 Additional Runtime - 14.28.29913
		  Microsoft Visual C++ 2019 X64 Minimum Runtime - 14.28.29913
		  ```
			- We could check if any know exploit exists for *7-Zip* or *XAMPP*.
		- Incomplete or flawed installation process can lead to installed applications to not be shown with the above mentioned method. Because of that it always a good idea to manually check for other software in the following folders.
			- `C:\Program Files`
			- `C:\Program Files (x86)`
			- The user's `Downloads` folder
- Running processes
	- We can use the [`Get-Process`](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-process?view=powershell-7.2) cmdlet in PowerShell.
	  ```powershell
	  PS C:\Users\dave> Get-Process
	  Get-Process
	  
	  Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName                                               
	  -------  ------    -----      -----     ------     --  -- -----------                                                
	       49      12      528       1152       0.03   2044   0 access
	  ...
	      477      49    17328      23904              6068   0 httpd
	      179      29     9608      19792              6824   0 httpd
	  ...                                                 
	      174      16   210620      29048              1752   0 mysqld
	  ...                                                  
	      825      40    75804      14404       5.91   6332   0 powershell
	  ...                                                
	      379      24     6864      30236              2272   1 xampp-control
	  ...
	  ```
		- Due to the active process named *xampp-control*, we can infer that both Apache *(httpd)* and MySQL *(mysqld)* were started through XAMPP.