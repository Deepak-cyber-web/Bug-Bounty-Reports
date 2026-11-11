# Title
Race Condition in Definition Voting Allows Multiple Votes from Single User

# Vulnerability Type
Race Condition

# Summary
The definition voting endpoint at `https://cablej.pwnedbox.site/vote.php` is vulnerable to a race condition. When 28 identical upvote requests for the same definition are sent simultaneously via Burp Suite Repeater (Send group in parallel), the server processes multiple votes concurrently before recording the first vote, returning `"success":true` on multiple parallel requests. The JSON response confirmed `"user_vote":"up"` and an inflated `"up"` count, proving multiple votes were registered from a single user session.

# Vulnerable Endpoint
POST `https://cablej.pwnedbox.site/vote.php`

# Steps to Reproduce
1. Log in to Urban Dictionary and navigate to any definition page.
2. Click the upvote (thumbs up) button and intercept the POST request in Burp Suite Proxy.
3. Send the intercepted request to Burp Repeater and duplicate it into a group of 28 identical tabs.
4. Click **Send group (parallel)** to fire all 28 requests simultaneously.
5. Observe the JSON responses — multiple tabs return `"success":true` with `"user_vote":"up"` and an inflated `"up"` vote count.

# Proof of Concept
<img width="1920" height="1040" alt="a1" src="https://github.com/user-attachments/assets/9b408f87-6c3d-469c-9671-3b4b2d031424" />
<img width="1920" height="1037" alt="a2" src="https://github.com/user-attachments/assets/01790226-8657-4a15-9fa3-de85b76c5f6e" />


# Impact
An attacker can artificially inflate or deflate vote counts on any definition, manipulating community rankings and surfacing or burying content unfairly at scale.

# Recommendation for fix
1. Use an atomic database transaction with a unique constraint on `(user_id, definition_id)` to ensure only one vote record exists per user per definition, rejecting duplicates at the DB level.
2. Implement a server-side distributed lock (e.g., Redis `SETNX`) scoped to `user_id + definition_id` before processing the vote to block concurrent requests from bypassing the duplicate check.
3. Validate the vote state server-side before incrementing the counter — check and set in a single atomic operation rather than separate read-then-write steps.
