# Write-up: Hidden Deep Into my Heart

## 1. Reconnaissance

I started the attack by scanning ports with the `nmap` tool, which revealed:

- **Port 22/tcp:** OpenSSH 8.9p1 (Ubuntu).
- **Port 5000/tcp:** HTTP service based on the Werkzeug 3.1.5 framework (Python 3.10.12).
    

![](<images/Pasted image 20260213225736.png>)

## 2. Enumeration and Information Leak (Robots.txt)

The first scan with the `gobuster` tool revealed two resources:

- `/console` (Status 400) – Werkzeug debugger.
- `/robots.txt` (Status 200).
    

Analyzing the `robots.txt` file turned out to be crucial. I found in it:

1. A hidden path blocked for bots: `/cupids_secret_vault/`.
    

![](<images/Pasted image 20260213225824.png>)

2. A comment left by the developer: `# cupid_arrow_2026!!!`.
    

![](<images/Pasted image 20260213225844.png>)

This comment was identified as a potential password (Sensitive Information Leak).

## 3. Directory Enumeration

After navigating to `/cupids_secret_vault/`, I received a message indicating that the search was not over yet.

![](<images/Pasted image 20260213225925.png>)

I ran a second `gobuster` scan, this time targeting the subdirectories of the hidden folder:

```Bash
gobuster dir -u http://10.67.176.96:5000/cupids_secret_vault/ -w /usr/share/wordlists/dirb/common.txt
```

![](<images/Pasted image 20260213225942.png>)

The scan revealed the endpoint: `/cupids_secret_vault/administrator`.

## 4. Exploitation (Broken Authentication)

At the indicated address, there was a login panel for "Cupid's Vault". I decided to use the credentials obtained from the `robots.txt` file.

![](<images/Pasted image 20260213230214.png>)

- **Attempt 1:** Login: `administrator`, Password: `cupid_arrow_2026!!!` (Failure).
    
- **Attempt 2:** Login: `admin`, Password: `cupid_arrow_2026!!!` (Success).
    

## 5. Getting the Flag

After successfully logging in, the system displayed a welcome message for the user `Cupid` along with the flag.

![](<images/Pasted image 20260213230230.png>)

**Flag:** `THM{l0v3_is_in_th3_r0b0ts_txt}`