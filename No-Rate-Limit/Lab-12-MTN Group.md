# Title
Missing Rate Limiting on 5-Digit OTP Verification Endpoint

# Vulnerability Type
Missing Rate Limiting / Broken Authentication

# Summary
The 5-digit OTP verification endpoint at `https://resend.pwnedbox.site/nim/submit.php` has no rate limiting. During testing, 101 sequential OTP values were submitted via Burp Suite Intruder and all returned `HTTP 303` with no throttling, lockout, or CAPTCHA triggered.

# Vulnerable Endpoint
`POST `https://resend.pwnedbox.site/nim/submit.php``

# Steps to Reproduce
1. Navigate to `https://resend.pwnedbox.site` and enter a valid phone number and NIN to reach the OTP verification step.
2. Intercept the OTP submission request in Burp Suite Proxy.
3. Send the intercepted request to Burp Intruder.
4. Set the `otp` parameter value as the payload position.
5. Configure payload type: Numbers, Sequential, From: `11111`, To: `11211`, Step: `1`, Min/Max integer digits: `5`.
6. Launch the Sniper attack.
7. Observe that all 101 requests return `HTTP 303` with no rate limiting enforced.

# Proof of Concept
<img width="1920" height="1028" alt="a1" src="https://github.com/user-attachments/assets/ae032359-502c-494c-b8fd-2f9b957beed4" />
<img width="1920" height="1032" alt="a2" src="https://github.com/user-attachments/assets/f90010cb-6fcd-441a-8656-839cb83f9428" />
<img width="1917" height="1021" alt="a3" src="https://github.com/user-attachments/assets/f79188e4-57f8-4a05-8f02-abd28691e6b1" />


# Impact
- An attacker can brute-force the 5-digit OTP (100,000 possible combinations) without any restriction, allowing full bypass of the NIN-to-SIM linking verification step.

# Recommendation for fix
- Lock the account or invalidate the session after 5 consecutive failed 2FA attempts
- Implement a time delay or cooldown between failed attempts
- Return `HTTP 429` when the rate limit is exceeded
