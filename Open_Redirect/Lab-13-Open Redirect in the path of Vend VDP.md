# Title
Open Redirect via Protocol-Relative Path Injection Directly in URL Path

# Vulnerability Type
Open Redirect

# Summary
The Vend POS application at `vendhq.pwnedbox.site` is vulnerable to an open redirect triggered directly in the **URL path** — no query parameter is involved. Appending `//bing.com` to the base URL causes the server to interpret the double-slash as a protocol-relative URL and redirect the user to an arbitrary external domain. This variant is particularly dangerous as it bypasses parameter-focused security filters entirely

# Vulnerable Endpoint
`https://vendhq.pwnedbox.site/`

# Steps to Reproduce
1. Open browser and navigate to:

```
   https://vendhq.pwnedbox.site//bing.com
```

2. Observe the Vend POS homepage loads briefly from the trusted domain
3. Browser silently redirects to external domain `bing.com`
4. No query parameter is used — the redirect is triggered purely via path manipulation

# Payload Used
```
https://vendhq.pwnedbox.site//bing.com
```

# Proof of Concept
<img width="949" height="512" alt="a1" src="https://github.com/user-attachments/assets/7ba3dc8a-f142-40d0-bde9-c36ebd07ae83" />
<img width="950" height="515" alt="a2" src="https://github.com/user-attachments/assets/1e500e49-4b07-4521-a50c-22d421cb9799" />
<img width="956" height="515" alt="a3" src="https://github.com/user-attachments/assets/fc5660ec-2ddb-49ef-b705-049fb738d166" />

# Impact
- Credential Phishing
- Session Token Theft

# Recommendation for fix
- Allowlist only internal/relative paths in `url` and `redir` params
- Reject any value containing `://` or external hostname

