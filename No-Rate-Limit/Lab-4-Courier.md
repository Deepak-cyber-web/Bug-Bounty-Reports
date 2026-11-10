# Title
No Rate Limit on User Registration Endpoint Allows Mass Account Creation and Email Enumeration

# Vulnerability Type
Missing Rate Limiting / Email Enumeration

# Summary
The user registration endpoint at `https://stress.pwnedbox.site/index.php` does not implement any rate limiting. An attacker can register unlimited accounts in rapid succession without any restriction. Additionally, the endpoint returns different HTTP response codes depending on whether an email already exists — `HTTP 200` for a new email and `HTTP 500` for an already registered one — allowing an attacker to enumerate valid registered emails.

# Vulnerable Endpoint
`POST https://stress.pwnedbox.site/index.php
Parameter: email`

# Steps to Reproduce
1. Navigate to `https://stress.pwnedbox.site/index.php` and open the registration form
2. Intercept the `POST /index.php` request using Burp Suite with a test email
3. Send the request to Burp Intruder and mark the numeric suffix of the email as payload position
4. Set payload type to **Numbers** (sequential, e.g. 2–52, step 1) — 51 requests total
5. Start the attack — no lockout or throttling is triggered
6. Observe: new emails return `HTTP 200` (account created), already registered emails return `HTTP 500` — confirming email enumeration
   
# Proof of Concept
<img width="1909" height="1038" alt="b1" src="https://github.com/user-attachments/assets/7fac7d4a-e0ca-4019-bc2e-feceeb36d86a" />
<img width="1914" height="1033" alt="b2" src="https://github.com/user-attachments/assets/d639578f-20ee-42cf-860e-c361d9db4b0f" />
<img width="1900" height="1024" alt="b3" src="https://github.com/user-attachments/assets/121331b9-df99-4aff-9cd9-7f9062a3149c" />
<img width="1915" height="1035" alt="b4" src="https://github.com/user-attachments/assets/28ecfc49-5b88-40f0-b716-33c09b0a7072" />
<img width="1899" height="1037" alt="b5" src="https://github.com/user-attachments/assets/84eaf691-2ef5-48ba-8a88-04eac672ab86" />


# Impact
- An attacker can create unlimited fake accounts without restriction and enumerate registered email addresses by observing the difference in HTTP response codes (200 vs 500).
# Recommendation for fix
- Implement rate limiting (e.g. max 5 attempts per IP per 15 minutes)
- Return HTTP 429 when the rate limit is exceeded
