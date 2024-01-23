public:: true

- [Privilege escalation](https://attack.mitre.org/tactics/TA0004) helps us to get the required privileges to:
	- Search for sensitive information in other users' home directories
	- Examine configuration files on the system
	- Extract password hashes with [[Mimikatz]]
- #+BEGIN_IMPORTANT
  Obtaining privileged access on every machine in a penetration test is rarely a useful or realistic goal. A skilled penetration tester's goal is therefore not to blindly attempt privilege escalation on every machine at any cost, but to identify machines where privileged access leads to further compromise of the client's infrastructure.
  #+END_IMPORTANT