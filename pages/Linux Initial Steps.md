public:: true
tags:: sudo, setuid

- **Check what command the user can run via `sudo`**
  ```bash
  sudo -l
  ```
	- For semi interactive shells use
	  ```bash
	  sudo -lS
	  ```
- **Look for unusual setuid executables**
  ```bash
  find / -perm -4000 2>/dev/null
  ```
- **Search for writable files in `/etc`**
  ```bash
  find /etc -type f -writable 2>/dev/null
  ```
- Search `passw` in webroot
  ```bash
  find . -type f -not -name "*.js" -not -name "*.map" -not -name "*css" -exec grep -nHi --color=always passw {} \;
  ```