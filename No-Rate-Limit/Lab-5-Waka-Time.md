# Title
No Rate Limit on Password Reset Endpoint Allows Email Bombing

# Vulnerability Type
Missing Rate Limiting on Password Reset

# Summary
The password reset endpoint at `https://request.pwnedbox.site/index.php` does not implement any rate limiting. An attacker can send unlimited password reset emails to any registered email address in rapid succession without being blocked or throttled. During testing, 51 requests were sent using Burp Suite Intruder and all returned `HTTP 200`, resulting in 50+ password reset emails being delivered to the target inbox within seconds.

# Vulnerable Endpoint
`POST https://request.pwnedbox.site/index.php
Parameter: email`

# Steps to Reproduce
1. Navigate to `https://request.pwnedbox.site/index.php#reset-section`
2. Enter a registered email and submit the password reset form
3. Intercept the `POST /index.php` request in Burp Suite
4. Send to Intruder, set a numeric payload position in the request (Sniper)
5. Set payload type to Numbers (sequential, 9–59, 51 requests) and start the attack
6. All 51 requests return `HTTP 200` — no lockout or throttling triggered
7. Check the target inbox — 50+  Reset your password” emails received

# Proof of Concept
<img width="1910" height="1038" alt="a1" src="https://github.com/user-attachments/assets/9d44fb08-f905-40ac-8102-9248e833d267" />
<img width="1916" height="1040" alt="a2" src="https://github.com/user-attachments/assets/6ea1f8ff-5e59-4bd2-9b64-0dcd7987d806" />
<img width="1909" height="1033" alt="a3" src="https://github.com/user-attachments/assets/a2d4b186-f529-4fa0-957b-3bde0e013408" />
<img width="1504" height="889" alt="a4" src="https://github.com/user-attachments/assets/08dd42ee-6975-4c50-967b-a8ec8dc0ef04" />

# Impact
- An attacker can flood any registered user’s inbox with unlimited password reset emails, causing email bombing and disrupting the user’s access to their account.

# Recommendation for fix
- Implement rate limiting (e.g. max 5 attempts per IP per 15 minutes)
- Return HTTP 429 when the rate limit is exceeded
