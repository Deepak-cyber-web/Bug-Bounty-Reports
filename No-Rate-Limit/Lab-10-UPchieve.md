# Title
No Rate Limit on Password Reset Endpoint Allows Email Bombing

# Vulnerability Type
Missing Rate Limiting

# Summary
The password reset endpoint at `https://ratelimit.pwnedbox.site/resetpassword.php` does not implement any rate limiting. An attacker can send unlimited password reset emails to any registered email address without being blocked or throttled. During testing, 51 requests were sent using Burp Suite Intruder and all returned `HTTP 200`, proving the target inbox can be flooded with reset emails in seconds.

# Vulnerable Endpoint
`POST https://ratelimit.pwnedbox.site/resetpassword.php
Target: POST /ratelimit/resetpassword.php (or /index.php)
Parameter: email`

# Steps to Reproduce
1. Navigate to `https://ratelimit.pwnedbox.site/resetpassword.php`
2. Enter a registered email and submit the password reset form
3. Intercept the `POST /resetpassword.php` request in Burp Suite
4. Send to Intruder, mark a numeric position in the request as the payload position
5. Set payload type to Numbers (sequential, 9–59, 51 requests) and start the attack
6. All 51 requests return `HTTP 200` — no lockout or throttling triggered
7. Target inbox receives 50+ “Password reset instructions” emails

# Proof of Concept
<img width="1916" height="1032" alt="a1" src="https://github.com/user-attachments/assets/1d584af8-3784-4dd8-a072-af3704bd9230" />
<img width="1920" height="1035" alt="a2" src="https://github.com/user-attachments/assets/bff9d8b9-fe58-4521-a13a-37c6cbd14616" />
<img width="1913" height="1017" alt="a3" src="https://github.com/user-attachments/assets/82834b80-ad4a-488c-91fa-91d4174e5aa7" />


# Impact
- An attacker can flood any registered user’s inbox with unlimited password reset emails, causing email bombing and disrupting the user’s account access.

# Recommendation for fix
- Limit password reset requests to 3 per email address per hour
- Return `HTTP 429` when the limit is exceeded
- Add CAPTCHA on the password reset **form**

