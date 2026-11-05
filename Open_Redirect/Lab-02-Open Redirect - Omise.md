# Title
Open Redirect via Protocol-Relative Double-Slash Path Injection on Omise Payment Gateway Domain

# Vulnerability Type
Open Redirect

# Summary
The Omise Developer Blog at `links.pwnedbox.site` is vulnerable to a path-based open redirect triggered by appending `//bing.com` directly to the base URL — no query parameter required. The server fails to normalize the path and interprets `//bing.com` as a protocol-relative external URL, silently redirecting users away from the trusted Omise domain. Given Omise’s role as a **payment gateway**, this vulnerability carries significantly elevated risk for financial phishing attacks.

# Vulnerable Endpoint
```
https://links.pwnedbox.site/
```

# Steps to Reproduce
1. Open browser and navigate to:

```
   https://links.pwnedbox.site//bing.com
```

2. Observe the legitimate Omise Developer Blog loads from trusted domain
3. No query parameter is involved — redirect triggered purely via path
4. Browser is redirected to external domain `bing.com`

# Payload Used
```
https://links.pwnedbox.site//bing.com
```

# Proof of Concept
<img width="951" height="515" alt="a1" src="https://github.com/user-attachments/assets/6c8248b6-1a20-4b7c-93d7-317aa052f093" />
<img width="947" height="515" alt="a2" src="https://github.com/user-attachments/assets/4b95fb76-25f1-49c8-b095-c6541ff9a6ca" />
<img width="952" height="512" alt="a3" src="https://github.com/user-attachments/assets/7dece944-6682-4aca-ad14-47982508f0bf" />

# Impact
- Credential Phishing
- Session Token Theft

# Recommendation for fix
- Allowlist only internal/relative paths in `url` and `redir` params
- Reject any value containing `://` or external hostname
