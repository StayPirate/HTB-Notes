public:: true

- #+BEGIN_TIP
  While hunting for users to impersonate, if a user is dormant (they have not changed their password or logged in recently) we will cause less interference and draw less attention if we take over that account.
  #+END_TIP
	- From `PowerView`:
	  ```powershell
	  Get-NetUser | select cn,pwdlastset,lastlogon
	  ```
- #+BEGIN_TIP
  If a user hasn't changed their password since a recent password policy change, their password may be weaker than the current policy. This might make it more vulnerable to password attacks.
  #+END_TIP
	- From `PowerView`:
	  ```powershell
	  Get-NetUser | select cn,pwdlastset
	  ```
- #+BEGIN_TIP
  *We may not always want to escalate our privileges right away.*
  
  It's important to establish a good foothold, and our goal at the very least should be to **maintain our access**. If we are able to compromise other users that have the same permissions as the user we already have access to, this allows us **to maintain our foothold**.
  If, for example, the password is reset for the user we originally obtained access to, or the system administrators notice suspicious activity and disable the account, we would still have access to the domain via other users we compromised.
  #+END_TIP
- {{embed ((655cf4a9-95ae-4eb8-be69-cd1fb9b251c6))}}