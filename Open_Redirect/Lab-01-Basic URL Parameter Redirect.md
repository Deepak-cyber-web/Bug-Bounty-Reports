# Title
Open Redirect via Unsanitized `url` Query Parameter on Dedicated Redirect Endpoint

# Vulnerability Type
Open-Redirect

# Summary
The application at `redirect.pwnedbox.site` exposes a dedicated `/` redirect endpoint that accepts a `url` parameter with no domain validation. Any user visiting a crafted link is silently forwarded to an arbitrary external domain. The existence of a purpose-built redirect endpoint with zero allowlist enforcement makes this a textbook open redirect vulnerability.

# Vulnerable Endpoint
```
https://redirect.pwnedbox.site/?url=
```

# Steps to Reproduce
1. Open browser and navigate to:

```
   https://redirect.pwnedbox.site/?url=https://bing.com
```

2. Observe the DevPulse page loads from trusted domain `redirect.pwnedbox.site`
3. The `url` parameter value is processed without validation
4. Browser is immediately redirected to `bing.com`

# Payload Used
```
?url=https://bing.com
```
# Proof of Concept
<img width="952" height="514" alt="a1" src="https://github.com/user-attachments/assets/5a5d5a46-bac1-4c9f-9e29-29f2d62d20cc" />
<img width="958" height="516" alt="a2" src="https://github.com/user-attachments/assets/fdbbda91-3797-49d8-8cc9-1d3b37642ea9" />
<img width="959" height="512" alt="a3" src="https://github.com/user-attachments/assets/eee00b85-3b0d-4ce1-8d5b-ddae160d4b80" />
<img width="961" height="516" alt="a4" src="https://github.com/user-attachments/assets/d63521f0-8d3d-44bb-ac98-7be11fc85595" />

# Impact
- Phishing via trusted DevPulse domain — victims expect external article links

# Recommendation for fix
- Allowlist only known publisher domains in the `url` parameter
