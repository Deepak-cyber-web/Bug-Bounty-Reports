# Title
Race Condition in CTF Flag Submission Allows Duplicate Point Accumulation

# Vulnerability Type
Race Condition

# Summary
The flag submission endpoint at `https://dropper.pwnedbox.site/submit.php` is vulnerable to a race condition. When 22 identical flag submission requests are sent simultaneously via Burp Suite Repeater (Send group in parallel), the server processes multiple requests concurrently before recording the first successful submission, awarding points multiple times for a single valid flag. The CTF dashboard confirmed the Trivial challenge was credited **10 times** (completion showed **10/1** with a “Race Triggered” badge) instead of the expected 1 point, accumulating 10/26 points from a single flag.

# Vulnerable Endpoint
POST `https://dropper.pwnedbox.site/submit.php`

# Steps to Reproduce
1. Log in to the Hacker101 CTF platform and navigate to a challenge.
2. Obtain the valid flag and submit it once normally to confirm it is accepted.
3. Intercept the flag submission POST request in Burp Suite Proxy.
4. Send the intercepted request to Burp Repeater and duplicate it into a group of 22 identical tabs.
5. Click **Send group (parallel)** to fire all 22 requests simultaneously.
6. Navigate back to the CTF dashboard.
7. Observe the challenge completion counter showing `10/1` with a “Race Triggered” badge and 10 points credited instead of 1.

# Proof of Concept\
<img width="1920" height="1041" alt="b1" src="https://github.com/user-attachments/assets/5f1884f9-f6a2-411a-9811-8b9021acf39a" />
<img width="1920" height="1036" alt="b2" src="https://github.com/user-attachments/assets/4f05a825-10ec-4d30-9a23-80d3190b5e0d" />
<img width="1920" height="1038" alt="b3" src="https://github.com/user-attachments/assets/68c6d7c2-dc8d-42a9-9e38-ed5821eb7aab" />


# Impact
An attacker can exploit this race condition to multiply points from a single valid flag submission, bypassing the one-flag-one-point limit and unfairly climbing the leaderboard or earning private invitations that require a points threshold.

# Recommendation for fix
1. Use an atomic database transaction with a unique constraint on `(user_id, flag_value)` to ensure a flag can only be credited once per user, rejecting duplicate inserts at the DB level.
2. Implement a server-side distributed lock (e.g., Redis `SETNX`) scoped to `user_id + flag_value` before processing the submission to block concurrent requests from passing the duplicate check simultaneously.
