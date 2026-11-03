# Title
Unlimited Pro Trial Access via Client-Controlled `trial_days` Parameter Tampering

# Vulnerability Type
Business Logic Flaw — Excessive Trust in Client-Supplied Trial Duration Parameter

# Summary
GigBoard's Pro plan upgrade flow sends the trial duration as a client-controlled parameter (`trial_days=7`) in the POST body when a user clicks "Start 7-Day Free Trial". The server accepts this value without validating it against the intended 7-day limit. By intercepting the request in Burp Suite and changing `trial_days=7` to `trial_days=90`, the attacker obtained a 90-day Pro trial instead of 7 days — with the trial end date confirmed as **01 Feb 2027** (90 days from exploit date). The UI states "One free trial per account" but the server enforces neither the duration cap nor the single-trial restriction at the request level.

# Vulnerable Endpoint
`POST https://gigboard.pwnedbox.site/upgrade/trial`

# Steps to Reproduce
1. Log in to GigBoard as user `dustylemur59803` (Free plan).
2. Navigate to the Upgrade page (`/upgrade`).
3. Enable Burp Suite Intercept (Proxy > Intercept ON).
4. Click **Start 7-Day Free Trial** on the Pro plan card.
5. Intercept the POST request to `/upgrade/trial`.
6. In the request body, locate the parameter `trial_days=7`.
7. Change it to `trial_days=90` and forward the request.
8. Navigate to Dashboard — Plan & Billing confirms:
    - Current plan: **Pro (trial)**
    - Trial ends: **01 Feb 2027** (90 days granted instead of 7).

# Payload Used
**Original POST body:**
```
_csrf=<token>&plan=pro&trial_days=7
```

**Modified POST body (exploit):**
```
_csrf=<token>&plan=pro&trial_days=90
```

Change: `trial_days` increased from `7` to `90`. Any arbitrary value is accepted — could be set to `365` or higher.

# Proof of Concept
<img width="1920" height="1043" alt="a1" src="https://github.com/user-attachments/assets/b74a7f60-c9dd-4133-be27-e34f7a3200fd" />
<img width="1920" height="1039" alt="a2" src="https://github.com/user-attachments/assets/ebd1e4cb-a050-4d7b-96b1-6e848d056596" />
<img width="1920" height="1034" alt="a3" src="https://github.com/user-attachments/assets/d7cb681d-1359-4a18-a631-4ef34d33f2c1" />


# Impact
- Direct Financial Loss
# Recommendation for fix
- **Never accept trial duration from the client.** The `trial_days` value must be hardcoded server-side (e.g., `TRIAL_DAYS = 7`) and never read from the request body.
