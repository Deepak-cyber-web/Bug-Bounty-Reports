# Title
Open Redirect via Quadruple-Slash (`////`) Path-Based Injection on Base Endpoint

# Vulnerability Type
Open Redirect

# Summary
The application at `stranger.pwnedbox.site` is vulnerable to a path-based open redirect triggered by appending `////bing.com/` directly to the base URL — no query parameter required. The server fails to normalize multiple leading slashes and interprets `////bing.com/` as a protocol-relative external URL, redirecting users to an arbitrary external domain. This variant bypasses single-slash (`/`) and double-slash (`//`) filters by using **four consecutive slashes**.

# Vulnerable Endpoint
```
https://stranger.pwnedbox.site/
```

# Steps to Reproduce
1. Open browser and navigate to:

```
   https://stranger.pwnedbox.site////bing.com/
```

2. No query parameter involved — redirect triggered via path only
3. Browser silently redirects to external domain `bing.com`

# Payload Used
```
https://stranger.pwnedbox.site////bing.com/
```

# Proof of Concept
<img width="1920" height="1035" alt="a1" src="https://github.com/user-attachments/assets/b6d1a6ae-bcbc-4fba-8ffa-49364526a6f3" />
<img width="1920" height="1041" alt="a2" src="https://github.com/user-attachments/assets/206b168b-0aef-4168-b447-ef76556c822c" />

# Impact
- Bypasses existing multi-slash filter — partial mitigation confirmed but incomplete

# Recommendation for fix
- Use server-level config: nginx `merge_slashes on` (note: only handles `//`, must add custom rule for `////`)
- Reject any path where normalized form begins with `//` or resolves to external host
