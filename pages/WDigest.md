public:: true

- Microsoft [docs](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc778868(v=ws.10)?redirectedfrom=MSDN)
- {{embed ((65663778-7eae-4a4d-a668-3bb7eff7d34d))}}
- After [security update](https://support.microsoft.com/en-us/topic/microsoft-security-advisory-update-to-improve-credentials-protection-and-management-may-13-2014-93434251-04ac-b7f3-52aa-9f951c14b649) [2871997](https://learn.microsoft.com/en-us/security-updates/securityadvisories/2016/2871997) WDigest does not store credentials in memory anymore.
	- `HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\SecurityProviders\WDigest`
		- If the `UseLogonCredential` value is set to **0**, WDigest will not store credentials in memory.
		- If the `UseLogonCredential` value is set to **1**, WDigest will store credentials in memory.
		- The `UseLogonCredential` value defaults to 0 when the registry entry is not present
		- By default in Windows 8.1 and Windows Server 2012 R2 and later versions, caching of credentials in memory for WDigest is disabled.