public:: true

- Additional information in [HackTricks](https://book.hacktricks.xyz/network-services-pentesting/pentesting-smtp)
- With [`smtp-user-enum`](https://pentestmonkey.net/tools/user-enumeration/smtp-user-enum) (pre-intalled in [Kali](https://www.kali.org/tools/smtp-user-enum/))
	- You first need to determine which of the three methods (`RCPT`, `VRFY`, `EXPN`) could work with [telnet](https://book.hacktricks.xyz/network-services-pentesting/pentesting-smtp#username-bruteforce-enumeration).
	- Then you can instruct `smtp-user-enum` to test all the names from a long wordlist.
	  ```bash
	  > smtp-user-enum -M RCPT -D domain-name.com -U ~/Downloads/names.txt -t 192.168.208.189                                                                       Starting smtp-user-enum v1.2 ( http://pentestmonkey.net/tools/smtp-user-enum )
	  
	   ----------------------------------------------------------
	  |                   Scan Information                       |
	   ----------------------------------------------------------
	  
	  Mode ..................... RCPT
	  Worker Processes ......... 5
	  Usernames file ........... /home/crazybyte/Downloads/names.txt
	  Target count ............. 1
	  Username count ........... 8607
	  Target TCP port .......... 25
	  Query timeout ............ 5 secs
	  Target domain ............ relia.com
	  
	  ######## Scan started at Tue Mar  5 11:34:30 2024 #########
	  192.168.208.189: bobby@domain-name.com exists
	  ######## Scan completed at Tue Mar  5 11:43:31 2024 #########
	  1 results.
	  
	  8607 queries in 541 seconds (15.9 queries / sec)
	  ```
		- *In this example I've used the [names.txt](https://github.com/huntergregal/wordlists/blob/master/names.txt) wordlist.*