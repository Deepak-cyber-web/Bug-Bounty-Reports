# Title
No Rate Limit on Login Endpoint Allows Unlimited Password Brute-Force

# Vulnerability Type
Broken Authentication / Missing Rate Limiting

# Summary
The login endpoint at `https://security-test.pwnedbox.site/index.php` does not implement any rate limiting, account lockout, or CAPTCHA. An attacker can send unlimited login attempts with different passwords for a known email address without being blocked or throttled. During testing, 73 requests were sent using Burp Suite Intruder with no lockout triggered. All wrong credentials returned HTTP 401 and a correct password returns HTTP 200, making automated brute-force attacks fully viable.

# Vulnerable Endpoint
`POST https://security-test.pwnedbox.site/index.php
Parameters: email, password`

# Steps to Reproduce
1. Navigate to `https://security-test.pwnedbox.site/index.php`
2. Intercept the login POST request using Burp Suite
3. Send the request to Burp Intruder and set the `password` field as the payload position
4. Load a password wordlist (73 passwords were used in this test)
5. Start the attack — all 73 requests complete with no lockout, delay, or CAPTCHA
6. Invalid passwords return HTTP 401; correct password returns HTTP 200

# Payload Used
- Attack Type: Sniper (Burp Suite Intruder)
- Payload Position: `password` parameter
- Payload Count: 73
- Target email: `partner@on-running.com`

# Proof of Concept
<img width="1915" height="1034" alt="a1" src="https://github.com/user-attachments/assets/7f70a5f8-9100-4e18-be18-a14d95ed3d7e" />
<img width="1918" height="1033" alt="a2" src="https://github.com/user-attachments/assets/d1131389-e4ed-4fce-8a1b-96313ebbb730" />
<img width="1900" height="1015" alt="a3" src="https://github.com/user-attachments/assets/942217cd-5d85-4f21-a784-09ee6f300940" />


# Impact
An attacker can brute-force account passwords without any restriction, leading to complete account takeover.

# Recommendation for fix
- Implement rate limiting (e.g. max 5 attempts per IP per 15 minutes)
- Return HTTP 429 when the rate limit is exceeded

