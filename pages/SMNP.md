- https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp
- Extended MIB are not installed by default due to licensing issue, but they can be manually installed as explained [here](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp#enumerating-snmp).
	- A blog post about how to get RCE from there is [here](https://mogwailabs.de/en/blog/2019/10/abusing-linux-snmp-for-rce/), and it may also be useful for privesc since `snmpd` runs as `root`.
- ```bash
  > snmpwalk -v 1 -c public 192.168.196.156 NET-SNMP-EXTEND-MIB::nsExtendOutputFull
  NET-SNMP-EXTEND-MIB::nsExtendOutputFull."reset-password" = STRING: 
  NET-SNMP-EXTEND-MIB::nsExtendOutputFull."reset-password-cmd" = STRING: "charlie:3PUKsX98BMupxxX" | chpasswd
  ```
	- More output can be fetched via
	  ```bash
	  snmpwalk -v 1 -c public 192.168.196.156 NET-SNMP-EXTEND-MIB::nsExtendObjects
	  ```