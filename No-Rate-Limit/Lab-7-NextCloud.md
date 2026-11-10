# Title
No Rate Limit on Newsletter Subscription Endpoint Allows Email Bombing

# Vulnerability Type
Missing Rate Limiting

# Summary
The newsletter subscription endpoint at `https://throttle.pwnedbox.site/index.php` does not implement any rate limiting. An attacker can submit unlimited subscription requests for the same email address without being blocked or throttled. During testing, 51 requests were sent using Burp Suite Intruder and all returned `HTTP 200`, resulting in 50 confirmation emails (“Finish your subscription to Nextcloud ne...”) delivered to the target inbox within seconds.

# Vulnerable Endpoint
`POST https://throttle.pwnedbox.site/index.php?p=subscribe&id=1
Parameters: htmlemail, list, listname, VerificationCodeX, email, emailconfirm, subscribe`

# Steps to Reproduce
1. Navigate to `https://throttle.pwnedbox.site` and fill in the newsletter subscription form with a test email
2. Intercept the `POST /index.php?p=subscribe&id=1` request in Burp Suite
3. Send to Intruder, mark a numeric position in the request as the payload position
4. Set payload type to Numbers (sequential, 9–59, 51 requests) and start the attack
5. All 51 requests return `HTTP 200` — no lockout or throttling triggered
6. Check the target inbox — 50 “Finish your subscription to Nextcloud newsletter” confirmation emails received

# Proof of Concept
<img width="1923" height="1039" alt="a1" src="https://github.com/user-attachments/assets/b7471d45-77bb-4111-9b28-bca511248477" />
<img width="1918" height="1040" alt="a2" src="https://github.com/user-attachments/assets/2e9b216e-4523-410f-b5a5-6ce005f6ae11" />
<img width="1914" height="1024" alt="a3" src="https://github.com/user-attachments/assets/72408eff-068e-4311-9cff-b7ee6ebd01a0" />
<img width="1919" height="1039" alt="a4" src="https://github.com/user-attachments/assets/15c56505-88c1-4e70-b702-8371def40a27" />


# Impact
- An attacker can flood any email address with unlimited Nextcloud newsletter confirmation emails, causing email bombing and inbox disruption without the victim’s consent.

# Recommendation for fix
- Limit subscription requests to 3 per email address per hour
- Return `HTTP 429` when the rate limit is exceeded
- Implement CAPTCHA on the subscription form to prevent automated submissions
