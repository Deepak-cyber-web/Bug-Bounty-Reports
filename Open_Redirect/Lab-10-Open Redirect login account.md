# Title
Open Redirect via `url` Parameter on Slack SSO Login Endpoint

# Vulnerability Type
Open Redirect

# Summary
The Slack SSO login page at `validation.pwnedbox.site` accepts an unsanitized `url` parameter that redirects users to arbitrary external domains after page load. Since this occurs on a **login page**, it is significantly more dangerous — an attacker can redirect victims to a cloned login page to harvest credentials.
# Vulnerable Endpoint
`https://validation.pwnedbox.site/?redir=llink?url=/messages`

# Steps to Reproduce
1. Open browser and visit:

```
   https://validation.pwnedbox.site/?redir=llink?url=https://bing.com
```

2. Observe the Slack “Sign in to Acme” login page loads from trusted domain
3. After redirect logic triggers, browser is sent to `bing.com`

# Payload Used
```
?redir=llink?url=https://bing.com
```

# Proof of Concept
<img width="958" height="517" alt="a1" src="https://github.com/user-attachments/assets/373e7756-dc25-4961-a761-0849d34f5eb8" />
<img width="958" height="517" alt="a2" src="https://github.com/user-attachments/assets/102121b4-2a1b-4790-aa31-c71f70e353e7" />


# Impact
- Credential Phishing
- Session Token Theft

# Recommendation for fix
- Allowlist only internal/relative paths in `url` and `redir` params
- Reject any value containing `://` or external hostname
