# Title
Race Condition in Folder Creation Allows Plan Limit Bypass

# Vulnerability Type
Race Condition

# Summary
The folder creation endpoint at `https://duststaff.pwnedbox.site/api/folders.php` is vulnerable to a race condition. The plan enforces a maximum of 10 folders per space, but when 23 identical folder creation requests are sent simultaneously via Burp Suite Repeater (Send group in parallel), multiple requests pass the plan-limit check concurrently before any single creation is recorded. This resulted in **19 folders being created against a limit of 10**, confirmed by the dashboard displaying “19 / 10 folders” with a “Limit Bypassed (19 Folders)” badge. Later requests correctly returned HTTP 403 with `"Your plan does not allow you to create more data sources."`, proving the check exists but is not atomic.

# Vulnerable Endpoint
POST `https://duststaff.pwnedbox.site/api/folders.php`

# Steps to Reproduce
1. Log in to Dust and navigate to Engineering Space → Knowledge → Folders.
2. Click “Add folder”, fill in a name and description, and intercept the POST request in Burp Suite Proxy.
3. Send the intercepted request to Burp Repeater and duplicate it into a group of 23 identical tabs.
4. Click **Send group (parallel)** to fire all 23 requests simultaneously.
5. Navigate back to the Folders page.
6. Observe the folder count showing **19/10** with a “Limit Bypassed” badge, confirming the plan limit was circumvented.

# Proof of Concept
<img width="1920" height="1031" alt="a1" src="https://github.com/user-attachments/assets/e0f29846-f6ca-45ca-8b5c-37e077443b40" />
<img width="1920" height="1042" alt="a2" src="https://github.com/user-attachments/assets/7e4f2406-ea86-4a16-a585-58d1b3a901b3" />
<img width="1920" height="1042" alt="a3" src="https://github.com/user-attachments/assets/13dee1d8-8df1-4d72-b57e-a93dd892e69b" />
<img width="1920" height="1040" alt="a4" src="https://github.com/user-attachments/assets/80aa6e35-79d8-4130-aed7-19fc491ff4ee" />


# Impact
An attacker can bypass paid plan restrictions to create unlimited folders for free, undermining the platform’s subscription model and causing unintended resource consumption.

# Recommendation for fix
1. Use an atomic database transaction with a row-level lock (e.g., `SELECT ... FOR UPDATE`) when checking and incrementing the folder count, so concurrent requests cannot both pass the limit check simultaneously.
2. Implement a server-side distributed lock (e.g., Redis `SETNX`) scoped to `workspace_id + space_id` before processing folder creation to serialize concurrent requests from the same account.

