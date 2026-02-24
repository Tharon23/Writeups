# Write-up: Signed Messages

## 1. Reconnaissance

Standard port scanning (`nmap`) revealed an HTTP service on port 5000 (Python/Flask). Directory enumeration (`gobuster`) discovered a key endpoint:

- `/debug`: A page containing system logs in developer mode.

![](<images/Pasted image 20260215170213.png>)

![](<images/Pasted image 20260215170223.png>)

## 3. Vulnerability Analysis

Analysis of the debug logs (`/debug`) revealed critical errors in the cryptography implementation (Broken Cryptography / Predictable RNG):

1. **Deterministic key generation:** RSA keys were not generated randomly, but based on a predictable seed dependent on the username.
    
    - Pattern: `{username}_lovenote_2026_valentine`

2. **Weak prime number generation algorithm:**
    - Prime number $P$ was derived from `SHA256(seed)`.
    - Prime number $Q$ was derived from `SHA256(seed + "pki")`.

3. **Improper key size:** Despite claiming to use "RSA-2048", using SHA-256 (which returns 256 bits) to generate components $P$ and $Q$ resulted in creating a key with a length of only **512 bits** ($256 + 256$), which is considered insecure.

## 4. Exploitation

The goal of the attack was to generate the private key for the user `admin` and forge a signature for their public welcome message.

### Step 1: Reconstructing the admin's private key

I wrote a Python script (using `pycryptodome` and `sympy` libraries) that replicated the server's flawed logic:

- A seed was created for the user `admin`.

- $P$ and $Q$ were calculated based on SHA-256 hashes.

- An RSA private key (512-bit) was constructed.


### Step 2: Signature Standard Issue (RSA-PSS)

Initial attempts to sign the message using the PKCS#1 v1.5 standard failed. Further analysis and tests showed that the application uses the **RSA-PSS (Probabilistic Signature Scheme)**.

Due to the non-standard small key size (512 bits), the standard salt length in PSS was too large, causing errors. It was necessary to manually calculate the maximum allowed salt length (`Max Salt`) in the exploit script.

### Step 3: Forgery (Signature Forgery)

I used the generated key and script to sign the exact content of the administrator's welcome message:

> _"Welcome to LoveNote! Send encrypted love messages this Valentine's Day. Your communications are secured with industry-standard RSA-2048 digital signatures."_

![](<images/Pasted image 20260215170443.png>)

### Step 4: Verification

I entered the obtained signature (Hex) on the `/verify` page, selecting `System Administrator (@admin)` as the sender.

![](<images/Pasted image 20260215170506.png>)

## 5. Outcome

The server verified the signature as valid ("Signature Valid"), which confirmed my identity as the administrator and revealed the flag.

**Flag:** `THM{PR3D1CT4BL3_S33D5_BR34K_H34RT5}`