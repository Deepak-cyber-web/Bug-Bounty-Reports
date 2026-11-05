# Title
Open Redirect Filter Bypass via Absolute URL in `returnTo` Parameter on `/logout` Endpoint

# Vulnerability Type
Open Redirect

# Summary
The Zaption application at `zaption.pwnedbox.site` uses a `returnTo` parameter on its `/logout` endpoint to redirect users after sign-out. The application appears to implement a filter accepting relative paths (e.g., `returnTo=/`) but fails to block absolute external URLs (e.g., `returnTo=https://bing.com`), allowing a complete filter bypass. An attacker can craft a logout link that redirects victims to a cloned Zaption login page — perfectly positioned for credential re-harvesting since the user just signed out.

# Vulnerable Endpoint
```
https://zaption.pwnedbox.site/logout?returnTo=
```
# Steps to Reproduce
1. Log in to `zaption.pwnedbox.site` (account: `gekko`)
2. Hover over **“Sign out”** — observe legitimate URL:

```
   https://zaption.pwnedbox.site/logout?returnTo=/
```

3. Navigate to crafted payload:

```
   https://zaption.pwnedbox.site/logout?returnTo=https://bing.com
```

4. Observe session is terminated on the trusted Zaption domain
5. Browser is redirected to external domain `bing.com` instead of `/`

# Payload Used
```
?returnTo=https://bing.com
```

# Proof of Concept
<img width="952" height="511" alt="b1" src="https://github.com/user-attachments/assets/4ccfaaf3-54e5-45e8-a583-529d0cd5c37d" />
<img width="955" height="514" alt="b2" src="https://github.com/user-attachments/assets/31b07259-0c9c-4f83-a0cf-a35f121f9a8c" />
<img width="953" height="512" alt="b3" src="https://github.com/user-attachments/assets/2e5169fe-3e5f-43eb-90c0-7b3ad722870e" />

# Impact
- Post-logout credential phishing

# Recommendation for fix
- Enforce strict relative-path-only check: `returnTo` must match `^/[^/\\]` regex
- Reject any value containing `://`, `//`, or external hostname
