public:: true
alias:: windows security questions

- Grzegorz Tworek [twitted](https://x.com/0gtweet/status/1787727775245193221) that Windows stores users' security questions in the registry and these can be locally dumped if you are logged in as Administrator in given target.
	- Later Adam Hassan [find out](https://hackback.zip/2024/05/08/Remotely-Dumping-Windows-Security-Questions-With-Impacket.html) that the the same can be achieved from remote via [SAMR](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-samr/4df07fab-1bbc-452f-8e92-7853a3c7e380), allowing to execute the attack over a large IP range very quickly without having to execute any code on the host.
- In NetExec `v1.3.0` the [`security-questions`](https://github.com/Pennyw0rth/NetExec/pull/295) module has been added to support [this feature](https://github.com/Pennyw0rth/NetExec/pull/295).
	- ```bash
	  netexec smb <host> -u <username> -p <password> -M security-questions
	  ```
	  ![Output of running security-questions NetExec module](https://hackback.zip//assets/images/blogs/security-questions/nxc-output.png)