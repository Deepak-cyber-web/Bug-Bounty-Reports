# Title
Open Redirect via `next` Parameter on Login Page Enables Post-Auth Credential Theft

# Vulnerability Type
Open Redirect

# Summary
The Weblate login endpoint at `immediately.pwnedbox.site` accepts an unsanitized `next` parameter that redirects users after authentication. Using a triple-slash (`///`) bypass, an attacker can redirect victims to any external domain after they sign in — including via GitHub, GitLab, or Google OAuth. This is especially critical as the redirect fires **after a successful login**, making it a direct vector for session token or credential theft.

# Vulnerable Endpoint
`https://immediately.pwnedbox.site/accounts/login/?next=`

# Steps to Reproduce
1. Open browser and navigate to:

```
   https://immediately.pwnedbox.site/accounts/login/?next=///bing.com
```

2. Observe the legitimate Weblate Sign In page loads on the trusted domain
3. The `next` parameter contains `///bing.com` — bypassing single/double slash filters
4. After the redirect logic processes, browser is sent to `bing.com`

# Payload Used
```
?next=///bing.com
```

# Proof of Concept
<img width="959" height="515" alt="a1" src="https://github.com/user-attachments/assets/a737426d-0c8d-4f2d-936c-160921e88f36" />
<img width="955" height="512" alt="a2" src="https://github.com/user-attachments/assets/2405e068-154c-4ec6-ba1d-40a320296807" />
<img width="956" height="517" alt="a3" src="https://github.com/user-attachments/assets/27d353d9-dcc8-4565-9aac-617201cd7f2f" />


# Impact
- Post-login session token leakage via `Referer` header to attacker domain

# Recommendation for fix

- Reject any `next` value not starting with a single `/` followed by a letter (e.g. regex: `^/[a-zA-Z]`)
-  Reject any value with external `://` or non-whitelisted hostnames
