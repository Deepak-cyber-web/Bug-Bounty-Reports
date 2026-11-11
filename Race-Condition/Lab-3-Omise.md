# Title
Race Condition in Team Member Invitation Allows Duplicate Invitations to Same Email

# Vulnerability Type
Race Condition

# Summary
The team member invitation endpoint at `https://condition.pwnedbox.site/team/memberships.php` is vulnerable to a race condition. When 21 identical invitation requests for the same email are sent simultaneously via Burp Suite Repeater (Send group in parallel), the server processes multiple requests before the duplicate check completes, resulting in the same email being added 8 times with “Pending Invitation” status. The application displayed a “Member Already Exists” error only after the race was won.

# Vulnerable Endpoint
POST `https://condition.pwnedbox.site/team/memberships.php`

# Steps to Reproduce
1. Log in to the Omise dashboard and navigate to Team & Access.
2. Click “Invite Member”, enter a target email address, select a role (e.g., Technical), and intercept the POST request in Burp Suite Proxy.
3. Send the intercepted request to Burp Repeater and duplicate it into a group of 21 identical tabs.
4. Click **Send group (parallel)** to fire all 21 requests simultaneously.
5. Navigate back to the Team Members page.
6. Observe the same email listed multiple times with “Pending Invitation” status.

# Proof of Concept
<img width="1920" height="1042" alt="c1" src="https://github.com/user-attachments/assets/d6880e15-3b0d-4e20-b55e-630912ae75b3" />
<img width="1920" height="1039" alt="c2" src="https://github.com/user-attachments/assets/19b89f94-9ccc-4788-9dea-e8261ac71edf" />
<img width="1920" height="1041" alt="c3" src="https://github.com/user-attachments/assets/e544b7aa-7c1c-43ad-9f31-b5c3cd4a1d1c" />

# Impact
An attacker can flood a victim’s inbox with multiple invitation emails and create duplicate pending membership records, causing confusion, potential privilege escalation if roles differ, and denial-of-service on the team management interface.

# Recommendation for fix
1. Use an atomic database constraint (unique index on `team_id + email`) so only one invitation record can exist per email per team, rejecting duplicates at the DB level.
2. Implement a server-side distributed lock (e.g., Redis `SETNX`) scoped to `team_id + email` before processing the invitation to prevent concurrent requests from passing the duplicate check simultaneously.
3. Apply idempotency tokens on the invitation form so re-submission or parallel submission of the same request is detected and rejected server-side.
