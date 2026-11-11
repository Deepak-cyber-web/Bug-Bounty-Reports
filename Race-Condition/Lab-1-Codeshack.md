# Title
Race Condition in OTP Resend Allows Resend Limit Bypass and Email Flooding

# Vulnerability Type
Race Condition

# Summary
The OTP resend endpoint at `https://counter.pwnedbox.site/resend.php` enforces a maximum of 3 resends per session, displayed as “3 resends left” on the UI. When 17 identical resend requests are sent simultaneously via Burp Suite Repeater (Send group in parallel), multiple requests pass the limit check concurrently before the counter is updated. The JSON response confirmed `"resend_count":7` and `"remaining_resends":0`, proving 7 OTP emails were sent instead of the allowed 3. Later requests correctly returned HTTP 429 with `"Maximum OTP resend limit reached (3 of 3)"`, confirming the check exists but is not atomic.

# Vulnerable Endpoint
POST `https://counter.pwnedbox.site/resend.php`

# Steps to Reproduce
1. Register or log in to CodeShack and reach the email verification page (`/verify.php`).
2. Click “Resend Code” and intercept the POST request to `/resend.php` in Burp Suite Proxy.
3. Send the intercepted request to Burp Repeater and duplicate it into a group of 17 identical tabs.
4. Click **Send group (parallel)** to fire all 17 requests simultaneously.
5. Observe that multiple tabs return HTTP 200 with `"success":true` and an inflated `resend_count` value (7 in testing).
6. Observe remaining tabs return HTTP 429 `"Maximum OTP resend limit reached (3 of 3)"`.

# Proof of Concept
<img width="1920" height="1040" alt="a1" src="https://github.com/user-attachments/assets/03c33c05-7594-46d9-a14d-5455477b5573" />
<img width="1920" height="1039" alt="a2" src="https://github.com/user-attachments/assets/e770748e-9d99-49cd-8974-594b2b68da78" />
<img width="1920" height="1040" alt="a3" src="https://github.com/user-attachments/assets/9ba61477-de6e-4f7e-acd1-1bfaa7d27c05" />
<img width="1920" height="1040" alt="a4" src="https://github.com/user-attachments/assets/526aef47-7c5f-4f8a-9e63-9e1fadf543f6" />

# Impact
An attacker can bypass the OTP resend limit to flood a victim’s inbox with excessive verification emails, causing email fatigue and potential denial-of-service on the target email account.

# Recommendation for fix
1. Use an atomic increment with a database-level check (e.g., `UPDATE ... WHERE resend_count < 3`) so the counter increment and limit validation happen in a single operation, rejecting any request that exceeds the limit at the DB level.
2. Implement a server-side distributed lock (e.g., Redis `SETNX`) scoped to the user session before processing each resend request to serialize concurrent submissions.
