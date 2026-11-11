# Title
Race Condition in Account Setup Survey Allows Unlimited Credit Redemption

# Vulnerability Type
Race Condition

# Summary
The account setup survey endpoint at `https://race-condition.pwnedbox.site/survey.php` is vulnerable to a race condition. When 20 identical survey submission requests are sent simultaneously via Burp Suite Repeater (Send group in parallel), the server processes multiple requests concurrently before recording the first completion, crediting $100 per request. The billing page confirmed $700 in credits from 7 successful parallel submissions instead of the expected $100 once.

# Vulnerable Endpoint
POST `https://race-condition.pwnedbox.site/survey.php`

# Steps to Reproduce
1. Log in to the Slack workspace and navigate to the Account Setup Survey (`/survey.php`).
2. Fill out the survey form and intercept the POST request in Burp Suite Proxy.
3. Send the request to Burp Repeater and duplicate it into a group of 20 identical tabs.
4. Click **Send group (parallel)** to fire all 20 requests simultaneously.
5. Navigate to `https://race-condition.pwnedbox.site/billing.php` and check billing history.
6. Observe multiple “Survey completed – Credited $100” entries for the same date.

# Proof of Concept
<img width="1914" height="1041" alt="b1" src="https://github.com/user-attachments/assets/0cdd0813-c534-4403-a6a0-cbda3506cbdd" />
<img width="1920" height="1044" alt="b2" src="https://github.com/user-attachments/assets/63edb07a-cf23-4088-98e7-147003be9155" />
<img width="1920" height="1038" alt="b3" src="https://github.com/user-attachments/assets/86399de4-9a0a-4ad1-a6dd-b3af86cefb00" />

# Impact
- An attacker can exploit this race condition to claim unlimited $100 credits by repeatedly submitting the one-time survey in parallel, causing significant financial loss to the platform.

# Recommendation for fix
1. Use an atomic database transaction with a unique constraint on `(user_id, survey_id)` to ensure the reward is recorded only once, even under concurrent requests.
2. Implement a server-side mutex or distributed lock (e.g., Redis `SETNX`) that blocks duplicate survey submissions for the same session until the first request completes.



