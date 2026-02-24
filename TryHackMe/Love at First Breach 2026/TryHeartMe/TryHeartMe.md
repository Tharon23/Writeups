# Write-up: TryHeartMe

## 1. Introduction and Reconnaissance

Upon entering the store page (`/shop`), we see the Valentine's assortment. Our goal is the hidden item **ValenFlag**.

![](<images/Pasted image 20260215173155.png>)

After logging in and checking the account balance in the `/account` tab, it turns out that as a newly registered user, we have **0 credits**. The store does not offer a legitimate way to top up the account ("Online top-ups are currently unavailable").

![](<images/Pasted image 20260215173235.png>)

Browser storage analysis (Developer Tools -> Storage -> Cookies) revealed that the application uses a session mechanism based on **JSON Web Token (JWT)**. The token is stored in a cookie named `tryheartme_jwt`.

![](<images/Pasted image 20260215173324.png>)

## 2. JWT Token Analysis

The token was copied and analyzed using the **CyberChef** tool. The token consists of three parts separated by dots (`Header.Payload.Signature`).

![](<images/Pasted image 20260215173426.png>)

**Decoded Payload:**

```JSON
{
  "email": "tharon@thm.com",
  "role": "user",
  "credits": 0,
  "iat": 1771171869,
  "theme": "valentine"
}
```

**Conclusions:**

1. **Insecure Storage:** The application stores key user state information (amount of credits, role) on the client side in the token, instead of verifying them in the database.
    
2. **Algorithm:** The header indicates the use of the **HS256** (HMAC-SHA256) algorithm, which means the token is signed with a symmetric key (password).
    

## 3. Vulnerability Detection

To forge the account balance, I had to generate a new signature for the modified data. To do this, the strength of the signing key (Secret Key) was tested.

![](<images/Pasted image 20260215173522.png>)

It turned out that the application had one of two (or both) critical vulnerabilities:

1. **Weak Secret:** The key used for signing is a trivial word: `secret`.
    
2. **Signature Verification Bypass:** Tests showed that the server also accepted tokens signed with a different key (e.g., `test`), suggesting that the signature verification mechanism on the server side is disabled or implemented incorrectly (CWE-347 vulnerability).
    

## 4. Exploitation (Step by Step)

Using **CyberChef**, a **JWT Forgery** attack was performed:

1. **Input:** Pasted the original JWT token.
    
2. **Operation:** Used the "JWT Decode" function to read, then modified the Payload manually.
    
3. **Modification:**
    
    - Changed `"credits": 0` to `"credits": 10000` (to afford the flag).
        
    - Changed `"role": "user"` to `"role": "admin"` (just in case the item was only available to admins).
        
4. **Re-sign:** Used the "JWT Sign" function with the key `secret`.
    
5. **Injection:** The new, forged token was pasted into the browser developer tools, replacing the session cookie value.
    
6. **Execution:** After refreshing the page, the account had administrator privileges and unlimited funds.

![](<images/Pasted image 20260215173634.png>)
    

## 5. Outcome

After successfully manipulating the token, the **ValenFlag** item became available for purchase. After clicking "Buy", the server generated a receipt containing the flag.

![](<images/Pasted image 20260215173645.png>)

**Obtained Flag:** `THM{v4l3nt1n3_jwt_c00k13_t4mp3r_4dm1n_sh0p}`