public:: true

- A key difference between Kerberos and [NTLM Authentication]([[AD NTLM Authentication]]) is that with the latter the client starts the authentication process with the application server itself. On the other hand, Kerberos client authentication involves the use of a domain controller in the role of a **Key Distribution Center ([KDC](https://en.wikipedia.org/wiki/Key_distribution_center))**. The client starts the authentication process with the KDC and not the application server.
  id:: 655a4269-4fa1-4988-b577-ad77f90064c0
- The *KDC* *(Key Distribution Center)* holds a database of the keys used in the authentication process and consists of two main parts:
	- The  *Authentication Server service* responsible for authenticating clients.
	  logseq.order-list-type:: number
	- The *Ticket Granting Service* provides tickets and [TGT](((655b1bc6-5c5d-4c70-9d2b-f3f3d6458cb9))) to the client systems.
	  logseq.order-list-type:: number
- User logon with Kerberos
  id:: 655a4269-b8f7-4f0f-b437-f04b362912d9
	- When a user logs in to their workstation, an *Authentication Server Request* (AS-REQ) is sent to the domain controller.
	  logseq.order-list-type:: number
	  id:: 655a1af9-9f7f-4089-8392-e6d816267e35
		- id:: 655b1c01-e1e1-4be0-b245-ff8c9482df38
		  #+BEGIN_PINNED
		  The **AS-REQ** *(Authentication Server Request)* contains:
		  #+END_PINNED
			- The timestamp, **encrypted with the NTLM hash of the user's password**
			  logseq.order-list-type:: number
			  id:: 655b158a-a666-41e0-8076-e59942a7bb20
			- The username
			  logseq.order-list-type:: number
	- Once the KDC received the request, it uses the user's password hash *(stored in the NTDS.dit file)* to decrypt the timestamp.
	  logseq.order-list-type:: number
		- In order to defeat potential [replay attacks](https://en.wikipedia.org/wiki/Replay_attack) attempts, it checks if the **same timestamp** was already submitted within a previous AS-REQ. If so the authentication fails.
		  logseq.order-list-type:: number
		- If the decryption succeeds and the provided timestamp was never submitted before, then the authentication succeeds.
		  logseq.order-list-type:: number
	- The domain controller replies to the client with an *Authentication Server Reply* (AS-REP).
	  logseq.order-list-type:: number
	  id:: 655a1fb1-47f5-446a-8813-e1a809f05a7b
		- id:: 655b1795-7a37-4e7f-b428-6c9a34ab2cbf
		  #+BEGIN_PINNED
		  The **AS-REP** *(Authentication Server Reply)* contains:
		  #+END_PINNED
			- A session key, **encrypted with the user's password hash**.
			  logseq.order-list-type:: number
			  id:: 655a201b-448f-45dc-a8b7-2c8978dde0c2
			- A *Ticket Granting Ticket* (TGT) **encrypted with a secret key (NTLM hash of the *krbtgt* account)** known only to the KDC. It cannot be decrypted by the client.
			  logseq.order-list-type:: number
			  id:: 655a4269-21dc-4947-a21d-5c89e404b561
				- id:: 655b1bc6-5c5d-4c70-9d2b-f3f3d6458cb9
				  #+BEGIN_PINNED
				  The **TGT** *(Ticket Granting Ticket)* contains:
				  #+END_PINNED
					- Information regarding the user
					  logseq.order-list-type:: number
					- Information regarding the domain
					  logseq.order-list-type:: number
					- A timestamp
					  logseq.order-list-type:: number
					- The IP address of the client
					  logseq.order-list-type:: number
					- The same [session key above](((655b1795-7a37-4e7f-b428-6c9a34ab2cbf))), but not encrypted with the user's password hash
					  logseq.order-list-type:: number
				- id:: 655a2173-161c-473c-bb1f-f4f6ee9aa3d9
				  #+BEGIN_NOTE
				  🕒 By default, the TGT will be valid for ten hours, after which a renewal occurs. This renewal does not require the user to re-enter their password.
				  #+END_NOTE
	- The authentication is now considered complete.
	  logseq.order-list-type:: number
- User request access to a domain service
	- ![Figure 2: Diagram of Kerberos Authentication](../assets/ad_kerbauth.png)
		- [Authentication Server Request](((655a4269-b8f7-4f0f-b437-f04b362912d9)))
		  logseq.order-list-type:: number
		- [Authentication server Replay](((655a4269-b8f7-4f0f-b437-f04b362912d9)))
		  logseq.order-list-type:: number
		- Now that the user is authenticated and has obtained a [session key](((655b1795-7a37-4e7f-b428-6c9a34ab2cbf))), he can craft *Ticket Granting Service Request* (TGS-REQ) to request access to domain services. As we [initially said](((655a4269-4fa1-4988-b577-ad77f90064c0))), this access request is filed against the KDC and not directly to the application server. Hence, the user (or client) will send the TGS-REQ to the KDC.
		  logseq.order-list-type:: number
		  id:: 655a4269-a509-429f-95ea-ce8b6582cf9c
			- #+BEGIN_PINNED
			  The **TGS-REQ** *(Ticket Granting Service Request)* contains:
			  #+END_PINNED
				- Username, **encrypted with the [session key](((655b1795-7a37-4e7f-b428-6c9a34ab2cbf)))**
				  logseq.order-list-type:: number
				- A timestamp, **encrypted with the [session key](((655b1795-7a37-4e7f-b428-6c9a34ab2cbf)))**
				  logseq.order-list-type:: number
				- The name of the domain service
				  logseq.order-list-type:: number
				- The [TGT](((655a4269-21dc-4947-a21d-5c89e404b561))) as it was provided by the KDC at authentication time. **Encrypted with a secret key (NTLM hash of the *krbtgt* account)** known only to the KDC
				  logseq.order-list-type:: number
		- The KDC can now validate the TGS-REQ and issue a *Ticket Granting Server Reply* (TGS-REP) to authorized the client to access the domain service.
		  logseq.order-list-type:: number
		  id:: 655a4269-b646-4744-a627-d2bb3f12a32d
			- The KDC decrypt the TGT using the secret key known only to the KDC (NTLM hash of the *krbtgt* account) and perform some validation checks.
			  logseq.order-list-type:: number
				- It extracts the session key and uses it to decrypt the username and timestamp of the request (TGS-REQ).
				  logseq.order-list-type:: number
				- At this point the KDC performs several checks:
				  logseq.order-list-type:: number
					- The TGT must have a [valid timestamp](((655a2173-161c-473c-bb1f-f4f6ee9aa3d9))).
					  logseq.order-list-type:: number
					- The username from the TGS-REQ has to match the username from the TGT.
					  logseq.order-list-type:: number
					- The client IP address needs to coincide with the TGT IP address.
					  logseq.order-list-type:: number
			- If the validation process succeeded, then the *ticket-granting service* (KDC) reply to the client with a *Ticket Granting Server Reply* (TGS-REP).
			  logseq.order-list-type:: number
				- #+BEGIN_PINNED
				  The **TGS-REP** *(Ticket Granting Server Reply)* contains:
				  #+END_PINNED
					- The name of the service for which access has been granted, **encrypted with the original [session key](((655b1795-7a37-4e7f-b428-6c9a34ab2cbf)))**.
					  logseq.order-list-type:: number
					- A new session key to be used between the client and the application server, **encrypted with the original [session key](((655b1795-7a37-4e7f-b428-6c9a34ab2cbf)))**.
					  logseq.order-list-type:: number
					  id:: 655a24b4-cedb-469a-91c2-8d5e46cbcd2c
					- A *service ticket* **encrypted with the NTLM hash of password of the service account associated to the service in question.** Hence, only the application server can decrypt it.
					  logseq.order-list-type:: number
					  id:: 655a24c7-b91e-4a45-8468-c565395f566e
						- id:: 655b24f3-a272-4dc0-814f-e7b3a4edf632
						  #+BEGIN_PINNED
						  The **service ticket** contains:
						  #+END_PINNED
							- The username of the user who requests access
							  logseq.order-list-type:: number
							  id:: 655a30d4-b9a7-4e83-94d1-750891d97ba6
							- The group memberships of the user who requests access
							  logseq.order-list-type:: number
							  id:: 655a30e3-6c13-444a-ad0a-b658667284e7
							- The newly created [session key](((655a24b4-cedb-469a-91c2-8d5e46cbcd2c)))
							  logseq.order-list-type:: number
							  id:: 655a30e9-90ec-4d1e-84b5-7ff17bc27a29
		- Now that the client has both the newly created [*session key*](((655a24b4-cedb-469a-91c2-8d5e46cbcd2c))) and the [*service ticket*](((655a24c7-b91e-4a45-8468-c565395f566e))), it can authenticate against the application server that hosts the domain service the user wants access to. In order to do that the client crafts an *Application Request* (AP-REQ) and send it to the application server.
		  logseq.order-list-type:: number
			- #+BEGIN_PINNED
			  The **AP-REQ** *(Application Request)* contains:
			  #+END_PINNED
				- The username, **encrypted the newly created [session key](((655a24b4-cedb-469a-91c2-8d5e46cbcd2c)))** *(the one associated with the service ticket)*.
				  logseq.order-list-type:: number
				- The timestamp, **encrypted the newly created [session key](((655a24b4-cedb-469a-91c2-8d5e46cbcd2c)))** *(the one associated with the service ticket)*.
				  logseq.order-list-type:: number
				- The [service ticket](((655a24c7-b91e-4a45-8468-c565395f566e))), **encrypted with the NTLM hash of password of the service account associated to the service in question**.
				  logseq.order-list-type:: number
		- The application server dcrypts the [*service ticket*](((655b24f3-a272-4dc0-814f-e7b3a4edf632))) with the NTML hash of the password associated to the service it is hosting. Please remind that this ticket was crated by the KDC and encrypted to make it readable only by the application server.
		  logseq.order-list-type:: number
			- The application server decrypts and extracts:
				- The username of the user who requests access
				  logseq.order-list-type:: number
				- The group memberships of the user who requests access
				  logseq.order-list-type:: number
				- The newly created [session key](((655a24b4-cedb-469a-91c2-8d5e46cbcd2c)))
				  logseq.order-list-type:: number
			- It can now use these information to check if the request is legit and if access to the service can be granted.
				- It checks if the username provided by the client in the AP-REQ matches.
				- It inspect the supplied group memberships from the service ticket and assigns appropriate permissions to the user.
			- If everything is fine, the user can now access the domain service.