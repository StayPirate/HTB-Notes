public:: true
alias:: nmap

- `nmap` doesn't contain [[WinRM]] ports in its top 1000 ports, hence they are not probed with the default options. Because of that it is important to ran a dedicated scan for the two WinRM ports.
  ```bash
  sudo nmap -sS 172.16.197.0/24 -p 5985,5986 --open
  ```