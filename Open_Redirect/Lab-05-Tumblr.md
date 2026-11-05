# Title
Open Redirect via `\@` Allowlist Bypass in `redirect_to` Parameter on Tumblr SSO Logout Endpoint

# Vulnerability Type
Open Redirect

# Summary
The Tumblr SSO endpoint at `sso.pwnedbox.site` uses a `redirect_to` parameter on logout and validates that the destination contains `www.tumblr.com`. However, this allowlist check is bypassable using a `\@` (backslash-AT) injection — the filter sees `www.tumblr.com` as the hostname, but the browser resolves `bing.com` as the actual destination. This is a **domain allowlist bypass**, not a simple open redirect — making it significantly more dangerous.

# Vulnerable Endpoint
```
https://sso.pwnedbox.site/?action=logout&redirect_to=
```

# Steps to Reproduce
1. Log in to `sso.pwnedbox.site` (Tumblr SSO)
2. Hover over **“Log out”** — observe legitimate URL in status bar:

```
   https://sso.pwnedbox.site/?action=logout&redirect_to=https://www.tumblr.com/dashboard
```

3. Navigate to crafted bypass payload:

```
   https://sso.pwnedbox.site/?action=logout&redirect_to=https://bing.com%5C%40www.tumblr.com
```

4. Session is terminated on trusted SSO domain
5. Browser redirects to `bing.com` — not `www.tumblr.com`

# Payload Used
```
redirect_to=https://bing.com%5C%40www.tumblr.com
```

# Proof of Concept
<img width="1913" height="1038" alt="b1" src="https://github.com/user-attachments/assets/df9ae4e7-99f2-4f41-b0d3-87928d50e490" />
<img width="1920" height="1037" alt="b2" src="https://github.com/user-attachments/assets/23c8f384-efed-4ca5-9465-0659435fa4eb" />
<img width="1920" height="1033" alt="b3" src="https://github.com/user-attachments/assets/0c71e9d2-3bf4-4080-821f-c1d5d1284835" />

# Impact
- Phishing

# Recommendation for fix
- Use an exact allowlist: `["https://www.tumblr.com/dashboard", ...]` or validate via `URL()` API
- Parse `redirect_to` using a proper URL parser — extract and compare **hostname only**, not full string match.
