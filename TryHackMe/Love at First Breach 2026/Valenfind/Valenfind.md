# Write-up: Valenfind

## 1. Reconnaissance

I started the attack by scanning ports with the `nmap` tool, which revealed:

- **Port 22/tcp:** SSH
- **Port 5000/tcp:** HTTP service (Werkzeug/Python Flask).
    

![](<images/Pasted image 20260213222323.png>)

Directory enumeration with the `gobuster` tool revealed standard entry points for a Flask application: `/login`, `/register`, `/dashboard`, and `/logout`.

![](<images/Pasted image 20260213222410.png>)

## 2. Session Analysis and SSTI Attempt

After registering an account, I analyzed the session cookie. It was a typical cryptographically signed Flask cookie. A brute-force attack attempt on the key (`SECRET_KEY`) using `flask-unsign` yielded no results (later code analysis showed that the key was generated randomly: `os.urandom(24)`).

![](<images/Pasted image 20260213223146.png>)

I also checked for an **SSTI (Server-Side Template Injection)** vulnerability by injecting `{{7*7}}` into the profile fields, but the data was rendered as static text, which initially blocked this path.

![](<images/Pasted image 20260213222451.png>)

## 3. LFI (Local File Inclusion) Vulnerability Detection

The key moment was discovering the profile theme change functionality. Traffic analysis in **Burp Suite** showed that the application fetches layout components via the endpoint: `GET /api/fetch_layout?layout=theme_classic.html`

![](<images/Pasted image 20260213222627.png>)

Injecting an incorrect file name revealed the absolute path on the server: `/opt/Valenfind/templates/components/`. I used the **Path Traversal** technique to break out of the intended directory and read the application's source code: `layout=../../app.py`

![](<images/Pasted image 20260213222650.png>)

## 4. Sensitive Data Exposure

Analyzing the read `app.py` file provided two critical pieces of information:

1. **ADMIN_API_KEY:** Hardcoded administrator key: `CUPID_MASTER_KEY_2024_XOXO`.

![](<images/Pasted image 20260213222713.png>)

2. **Hidden Endpoint:** `/api/admin/export_db`, which allows downloading the entire SQLite database, provided a valid token is sent in the `X-Valentine-Token` header.

![](<images/Pasted image 20260213222805.png>)

## 5. Final Exploitation and Database Exfiltration

I used the obtained API key to impersonate the administrator and download the `cupid.db` database. I used the `curl` tool for this:


```Bash
curl -H "X-Valentine-Token: CUPID_MASTER_KEY_2024_XOXO" [http://10.66.189.189:5000/api/admin/export_db](http://10.66.189.189:5000/api/admin/export_db) --output valenfind.db
```

![](<images/Pasted image 20260213222906.png>)

After saving the response as a `.db` file, I analyzed the `users` table using `sqlite3`.

![](<images/Pasted image 20260213223015.png>)

## 6. Getting the Flag (Root Cause)

The flag was located in the user record with the login `cupid` (System Administrator), in the `address/bio` column.

![](<images/Pasted image 20260213223037.png>)

**Flag:** `THM{v1be_c0ding_1s_n0t_my_cup_0f_t3a}`