public:: true

- #+BEGIN_QUOTE
  Microsoft [docs](https://learn.microsoft.com/en-us/windows-server/security/credentials-protection-and-management/protected-users-security-group)
  Introduced in Windows 8.1 and Windows Server 2012 R2
  Backported to Windows 7, Windows Server 2008 R2 and Windows Server 2012 via [MSA-2871997](https://learn.microsoft.com/en-us/security-updates/SecurityAdvisories/2016/2871997)
  #+END_QUOTE
- Members of this group automatically have **non-configurable protections** applied to their accounts. Membership in the Protected Users group is meant to be restrictive and proactively secure by default. The only method to modify these protections for an account is to remove the account from the security group.
- #+BEGIN_CAUTION
  Accounts for services and computers should never be members of the Protected Users group. This group provides incomplete protection anyway, because the password or certificate is always available on the host. Authentication will fail with the error "the user name or password is incorrect" for any service or computer that is added to the Protected Users group.
  #+END_CAUTION