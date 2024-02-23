public:: true

- #+BEGIN_NOTE
  Never underestimate the laziness of users when it comes to passwords and sensitive information.
  #+END_NOTE
- #+BEGIN_TIP
  The following steps should be performed for every users you get access to. Since each user will have different permissions, then different results should be expected.
  #+END_TIP
- Sensitive information may be stored in
	- Meeting notes
	- Configuration files
	- Text files
	- Onboarding documents
- Search files for its extension
	- In a [previous example](((65afa6b7-48a0-4e1e-bed6-26b11224ff1e))) we found out that KeePass was installed in a machine we broke into. Let's search for `.kdbx` files across the whole filesystem *(C:\)* with the [`Get-ChildItem`](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-childitem?view=powershell-7.4) cmdlet in PowerShell.
	  ```powershell
	  PS C:\Users\dave> Get-ChildItem -Path C:\ -Include *.kdbx -File -Recurse -ErrorAction SilentlyContinue
	  ```
	- Let's try to search for `.txt` or `.ini` configuration files within the XAMPP installation folder *(C:\xampp)*.
	  ```powershell
	  PS C:\Users\dave> Get-ChildItem -Path C:\xampp -Include *.txt,*.ini -File -Recurse -ErrorAction SilentlyContinue
	  Get-ChildItem -Path C:\xampp -Include *.txt,*.ini -File -Recurse -ErrorAction SilentlyContinue
	  ...
	  Directory: C:\xampp\mysql\bin
	  
	  Mode                 LastWriteTime         Length Name                                               
	  ----                 -------------         ------ ----                                               
	  -a----         6/16/2022   1:42 PM           5786 my.ini
	  ...
	  Directory: C:\xampp
	  
	  Mode                 LastWriteTime         Length Name                                              
	  ----                 -------------         ------ ----                                                                 
	  -a----         3/13/2017   4:04 AM            824 passwords.txt
	  -a----         6/16/2022  10:22 AM            792 properties.ini     
	  -a----         5/16/2022  12:21 AM           7498 readme_de.txt 
	  -a----         5/16/2022  12:21 AM           7368 readme_en.txt     
	  -a----         6/16/2022   1:17 PM           1200 xampp-control.ini 
	  ```
		- **my.ini** is the configuration file for MySQL, which probably contains credentials to authenticate to the MySQL server.
		- **passwords.txt** contains the default passwords for the different XAMPP components.
	- Let's search for documents *(*.txt,*.pdf,*.xls,*.xlsx,*.doc,*.docx)* within the user home directory *(C:\Users\dave\)*.
	  ```powershell
	  PS C:\Users\dave> Get-ChildItem -Path C:\Users -Include *.txt,*.pdf,*.xls,*.xlsx,*.doc,*.docx,*.kdbx,*.kdb,*.log -File -Recurse -ErrorAction SilentlyContinue
	  Get-ChildItem -Path C:\Users -Include *.txt,*.pdf,*.xls,*.xlsx,*.doc,*.docx,*.kdbx,*.kdb,*.log -File -Recurse -ErrorAction SilentlyContinue
	  
	      Directory: C:\Users\dave\Desktop
	  
	  
	  Mode                 LastWriteTime         Length Name                                                                 
	  ----                 -------------         ------ ----                                                                 
	  -a----         6/16/2022  11:28 AM            339 meeting-minutes.txt
	  ```
- To log as a different user after we retrieve his credentials (either plaintext password or its hash) we could:
	- If the user is a member of the `Remote Desktop Users` group and RDP port is reachable we can use RDP.
	- If the user is a member of the `Remote Management Users` or `Administrators` group and either port 5986 or 5985 are open, then we can use [[WinRM]].
	- If the user is a member of the `Administrators` group and port 135 is open we could use [[DCOM]].
	- If port 135 is open [[WMI]] ([deprecated](((65a7a0b2-1d28-4245-9f26-d094d4c26a84)))) can be tried to get a remote shell.
	- If the user has [`Log on as a batch job`](https://learn.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/log-on-as-a-batch-job) access right, you can schedule a task to execute commands.
	- If the user has an active session and port 445 is open, then you can use [[psexec]].
	- If you already have access to a GUI (e.g. via RDP) as another user, the you can use the [`runas`](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc771525(v=ws.11)) command. It can be used with local or domain accounts, but plaintext password is required.
		- ```cmd
		  C:\Users\steve> runas /user:backupadmin cmd
		  Enter the password for backupadmin:
		  Attempting to start cmd as user "CLIENTWK220\backupadmin" ...
		  ```
		  If the right password is entered a new command line window will appear.
- More trick to try [here](http://michalszalkowski.com/security/windows/privilege-escalation/).