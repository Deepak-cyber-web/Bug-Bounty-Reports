# Title
No Rate Limit on 2FA Code Verification Endpoint Allows Brute-Force of OTP

# Vulnerability Type
Missing Rate Limiting on 2FA Verification

# Summary
The 2FA verification endpoint at `https://rate.pwnedbox.site/users/testqr.php` does not implement any rate limiting or account lockout. An attacker who has obtained valid credentials can brute-force the 4-digit TOTP code without being blocked or throttled. During testing, 101 sequential numeric codes were submitted using Burp Suite Intruder and all returned `HTTP 302` with no lockout triggered. Since a 4-digit code has only 10,000 possible values, an attacker can enumerate all combinations and bypass 2FA completely.

# Vulnerable Endpoint
`POST https://rate.pwnedbox.site/users/testqr.php
API: POST /rate/users/testqr.php`

# Steps to Reproduce
1. Log in to `https://rate.pwnedbox.site` with valid credentials and reach the 2FA verification page
2. Enter any 4-digit code and intercept the `POST /users/testqr.php` request in Burp Suite
3. Send to Intruder, mark the `gauth_token` value as the payload position
4. Set payload type to Numbers (sequential, 5790–5890, 101 requests, 4-digit format) and start the attack
5. All 101 requests return `HTTP 302` — no lockout or throttling triggered
6. A correct code would redirect to `dashboard.php`, making success detection trivial

# Proof of Concept
<img width="1913" height="1042" alt="a1" src="https://github.com/user-attachments/assets/6b86768e-446c-4c46-8c02-3b312e05c127" />
<img width="1917" height="1041" alt="a2" src="https://github.com/user-attachments/assets/ffd85f15-2e2f-4a72-8682-711ddce71bd3" />
<img width="1917" height="1027" alt="a3" src="https://github.com/user-attachments/assets/f2817339-0177-4b98-96a8-1dcedb637323" />

# Impact
- An attacker with valid credentials can brute-force all 10,000 possible 4-digit 2FA codes without any restriction, completely bypassing two-factor authentication and gaining full account access.

# Recommendation for fix
- Lock the account or invalidate the session after 5 consecutive failed 2FA attempts
- Implement a time delay or cooldown between failed attempts
- Return `HTTP 429` when the rate limit is exceeded

