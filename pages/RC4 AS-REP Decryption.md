tags:: CVE-2022-33647,CVE-2022-33679

- The whole attack is based on the initial [research](https://googleprojectzero.blogspot.com/2022/10/rc4-is-still-considered-harmful.html) by [James Forshaw](https://twitter.com/tiraniddo) from [Project Zero](https://googleprojectzero.blogspot.com/p/about-project-zero.html).
- The two techniques described below got assigned [CVE-2022-33647](https://nvd.nist.gov/vuln/detail/CVE-2022-33647) and [CVE-2022-33679](https://nvd.nist.gov/vuln/detail/CVE-2022-33679).
- #+BEGIN_NOTE
  Kerberos RC4-MD4 downgrade attack
  
  The main idea is based on the fact that the key stream used for the timestamp must be the same as used in the response to encrypt the session key.
  #+END_NOTE
- TODO get inspiration from this [blog post](https://www.chudamax.com/posts/kerberos-102-overview/#rc4-as-rep-decryption-cve-2022-33647-cve-2022-33679)
  background-color:: pink