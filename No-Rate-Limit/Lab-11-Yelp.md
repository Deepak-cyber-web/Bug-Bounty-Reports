# Title
No Rate Limit on Resend Confirmation Email Endpoint Allows Email Bombing

# Vulnerability Type
Missing Rate Limiting

# Summary
The resend confirmation email endpoint at `https://reset.pwnedbox.site/welcome/resend_confirmation.php` has no rate limiting. During testing, 51 requests were sent via Burp Suite Intruder and all returned `HTTP 302` with no throttling, flooding the target inbox with 50+ confirmation emails.

# Vulnerable Endpoint
`POST https://reset.pwnedbox.site/welcome/resend_confirmation.php
Target: POST /reset/welcome/resend_confirmation.php
Parameters: csrftok, return_url, email`

# Steps to Reproduce
1. Log in to `https://reset.pwnedbox.site` as a business owner and navigate to the Inbox section
2. Click “Re-send email now” and intercept the `POST /welcome/resend_confirmation.php` request in Burp Suite
3. Send to Intruder, mark a numeric position in the request as the payload position
4. Set payload type to Numbers (sequential, 9–59, 51 requests) and start the attack
5. All 51 requests return `HTTP 302` — no lockout or throttling triggered
6. Target inbox receives 50+ “Confirmation email has been resent” emails


# Proof of Concept
<img width="1920" height="1031" alt="a1" src="https://github.com/user-attachments/assets/4664a662-7cd4-479d-b6c3-4da474a4dff3" />
<img width="1916" height="1032" alt="a2" src="https://github.com/user-attachments/assets/17d42cc6-7d57-421d-bd2a-a73604aa9ec6" />
<img width="1920" height="1045" alt="a3" src="https://github.com/user-attachments/assets/400607c5-de97-4e18-b65e-778717f43f0b" />

# Impact
- An attacker can flood any Yelp business owner’s inbox with unlimited confirmation emails, causing email bombing and disrupting their business account access.

# Recommendation for fix
- Limit resend confirmation requests to 3 per account per hour
- Return `HTTP 429` when the limit is exceeded
- Add a cooldown timer on the “Re-send email now” button both client-side and server-side
