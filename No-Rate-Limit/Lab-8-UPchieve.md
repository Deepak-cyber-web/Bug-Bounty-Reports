# Title
No Rate Limit on Contact Support Endpoint Allows Message Flooding

# Vulnerability Type
Missing Rate Limiting

# Summary
The contact support endpoint at `https://limit.pwnedbox.site/index.php` does not implement any rate limiting. An attacker can submit unlimited support messages without being blocked or throttled. During testing, 51 requests were sent using Burp Suite Intruder and all returned `HTTP 200`, proving the support team’s inbox can be flooded with automated messages from any email address.

# Vulnerable Endpoint
`POST https://limit.pwnedbox.site/index.php
API: POST /limit/api-public/contact/send
Parameters: email, topic, message`

# Steps to Reproduce
1. Navigate to `https://limit.pwnedbox.site/index.php#contact-section`
2. Fill in the contact form with a test email, topic, and message and submit
3. Intercept the `POST /index.php` request in Burp Suite
4. Send to Intruder, mark a numeric position as the payload position
5. Set payload type to Numbers (sequential, 9–59, 51 requests) and start the attack
6. All 51 requests return `HTTP 200` — no lockout or throttling triggered


# Proof of Concept
<img width="1907" height="1035" alt="a1" src="https://github.com/user-attachments/assets/fd4c9e16-c3b2-4bce-8d23-25a4f9953712" />
<img width="1909" height="1028" alt="a2" src="https://github.com/user-attachments/assets/d2ec3d0d-bca3-40b3-9878-b102f8aeb412" />
<img width="1906" height="1021" alt="a3" src="https://github.com/user-attachments/assets/917a1eaa-d976-4fc9-92c6-fc8258aca192" />


# Impact
- An attacker can flood the UPchieve support team’s inbox with unlimited automated messages, disrupting support operations and potentially hiding legitimate student requests.

# Recommendation for fix
- Limit contact form submissions to 3–5 per IP per hour
- Return `HTTP 429` when the limit is exceeded
- Add CAPTCHA on the contact form to block automated submissions

