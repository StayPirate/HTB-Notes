public:: true

- ```
      __             __               __
     / /_____  _____/ /_  _______  __/ /____
    / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
   / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
  /_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/
  
  Version: dev (bc1d606) - 11/15/20 - Ronnie Flathers @ropnop
  ```
- Quickly bruteforce and enumerate valid Active Directory accounts through Kerberos Pre-Authentication.
- id:: 655cbd1e-9c3e-4f5c-af2e-247535fac289
  #+BEGIN_NOTE
  The main advantage is that it **only uses two UDP frames** to determine whether the password is valid as it sends only an AS-REQ and examines the response, making it the faster online brute-force technique.
  
  It's also *kinda stealthier* than other methods since **pre-authentication failures do not trigger** "*An account failed to log on*" event 4625. However, the [KDC](((655a4269-4fa1-4988-b577-ad77f90064c0))) will log every [failed pre-authentication attempt](https://social.technet.microsoft.com/wiki/contents/articles/23559.kerberos-pre-authentication-why-it-should-not-be-disabled.aspx) by increasing the *badpwdcount* attribute for any incorrect `PA-ENC-TIMESTAMP` attempts.
  #+END_NOTE
- Official [repository](https://github.com/ropnop/kerbrute)
- Download binaries 💾 from official [releases](https://github.com/ropnop/kerbrute/releases)
- It's written in Go so it can be run from both Windows and **Linux**
- The working principle is similar to what [`kinit`](https://web.mit.edu/kerberos/krb5-1.12/doc/user/user_commands/kinit.html) does *([src](https://github.com/krb5/krb5/blob/master/src/clients/kinit/kinit.c))*. It sends [AS-REQ](((655b1c01-e1e1-4be0-b245-ff8c9482df38)))s to the domain controller and examines the response.
- Subcommands
	- **bruteuser** - Bruteforce a single user's password from a wordlist
	- **bruteforce** - Read username:password combos from a file or stdin and test them
	- **passwordspray** - Test a single password against a list of users
	- **userenum** - Enumerate valid domain usernames via Kerberos
- #+BEGIN_WARNING
  If you receive a network error, make sure that the encoding of `usernames.txt` is ANSI.
  #+END_WARNING
- Usage
	- ```
	  $ ./kerbrute -h
	  
	      __             __               __
	     / /_____  _____/ /_  _______  __/ /____
	    / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
	   / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
	  /_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/
	  
	  Version: dev (bc1d606) - 11/15/20 - Ronnie Flathers @ropnop
	  
	  This tool is designed to assist in quickly bruteforcing valid Active Directory accounts through Kerberos Pre-Authentication.
	  It is designed to be used on an internal Windows domain with access to one of the Domain Controllers.
	  Warning: failed Kerberos Pre-Auth counts as a failed login and WILL lock out accounts
	  
	  Usage:
	    kerbrute [command]
	  
	  Available Commands:
	    bruteforce    Bruteforce username:password combos, from a file or stdin
	    bruteuser     Bruteforce a single user's password from a wordlist
	    help          Help about any command
	    passwordspray Test a single password against a list of users
	    userenum      Enumerate valid domain usernames via Kerberos
	    version       Display version info and quit
	  
	  Flags:
	        --dc string          The location of the Domain Controller (KDC) to target. If blank, will lookup via DNS
	        --delay int          Delay in millisecond between each attempt. Will always use single thread if set
	    -d, --domain string      The full domain to use (e.g. contoso.com)
	        --downgrade          Force downgraded encryption type (arcfour-hmac-md5)
	        --hash-file string   File to save AS-REP hashes to (if any captured), otherwise just logged
	    -h, --help               help for kerbrute
	    -o, --output string      File to write logs to. Optional.
	        --safe               Safe mode. Will abort if any user comes back as locked out. Default: FALSE
	    -t, --threads int        Threads to use (default 10)
	    -v, --verbose            Log failures and errors
	  
	  Use "kerbrute [command] --help" for more information about a command.
	  ```