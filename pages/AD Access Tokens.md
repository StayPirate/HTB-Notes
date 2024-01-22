public:: true

- An [access token](https://learn.microsoft.com/en-us/windows/win32/secauthz/access-tokens) is an object that describes the [security context](https://learn.microsoft.com/en-us/windows/win32/wsw/security-context) of a process or thread.
- Once a user successfully logs on gets assigned an *access token*.
- #+BEGIN_IMPORTANT
  Every time the user executes a new process or thread his token is copied over it.
  That ensures every new process runs with the same privileges of the user who executed it.
  #+END_IMPORTANT
- The system checks the *access token* when a thread interacts with a [securable object](https://learn.microsoft.com/en-us/windows/win32/secauthz/securable-objects) or tries to perform a system task that requires privileges.
- *Access tokens* contain the following information
	- The [SID](((65ae6139-ddd3-44a1-906e-f0302a9151e7))) for the user's account
	- SIDs for the groups of which the user is a member
	- A list of the privileges held by either the user or the user's groups
	- The SID for the primary group
	- The [integrity SID]([[Mandatory Integrity Control]]) ([one or more](https://learn.microsoft.com/en-us/windows/win32/secauthz/mandatory-integrity-control#integrity-labels))
	- The soThe IL for aurce of the access token
	- Whether the token is a primary or [impersonation token](https://learn.microsoft.com/en-us/windows/win32/secauthz/client-impersonation)
	- An optional list of [restricting SIDs](https://learn.microsoft.com/en-us/windows/win32/secauthz/restricted-tokens)
- By default, the system uses the primary token when a thread of the process interacts with a securable object.
- A thread can also have an impersonation token assigned.
	- Impersonation tokens are used to provide a different *security context* than the process that owns the thread.
	  This means that the thread interacts with objects on behalf of the impersonation token instead of the primary token of the process.
	- A thread that is impersonating a client has both a primary token and an impersonation token.