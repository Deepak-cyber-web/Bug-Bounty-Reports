# Title
Open Redirect via Double-Slash Protocol-Relative URL on Base Endpoint

# Vulnerability Type
Open Redirect

# Summary
The application fails to validate redirect destinations when a double-slash (`//`) is appended directly to the base URL. This allows an attacker to redirect users to any arbitrary external domain while the URL appears to originate from the trusted `affirm.pwnedbox.site` domain.

# Vulnerable Endpoint
`https://affirm.pwnedbox.site/`

# Steps to Reproduce
1. Open a browser
2. Navigate to the following URL:

```
   https://affirm.pwnedbox.site//bing.com
```

3. Observe that the browser redirects away from `affirm.pwnedbox.site` to `bing.com`

# Payload Used
```
https://affirm.pwnedbox.site//bing.com
```

# Proof of Concept
<img width="959" height="506" alt="a1" src="https://github.com/user-attachments/assets/4d70ea57-0259-49a6-8c64-cdb235148385" />
<img width="958" height="506" alt="a2" src="https://github.com/user-attachments/assets/fd1a2f49-f158-481b-b65e-bab2062f4fd8" />
<img width="960" height="515" alt="a3" src="https://github.com/user-attachments/assets/5300d792-e4a3-4580-8b3d-193f50d62b68" />


# Impact
- Phishing via trusted Affirm domain link

# Recommendation for fix
- Enforce strict same-origin or allowlist-based redirect validation
- Reject URLs containing `//`, `\/`, or external hostnames
- Use relative paths only for redirects
