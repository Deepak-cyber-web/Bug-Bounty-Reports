# Title
Open Redirect via Unsanitized `url` Parameter on Outbound Link Tracking Endpoint — Semrush

# Vulnerability Type

# Summary
The Semrush application at `out.pwnedbox.site` uses a `?url=` parameter to route users from internal “Visit Page” buttons to external destinations (e.g., semrush.com feature pages). This same parameter accepts arbitrary external URLs without domain validation, allowing an attacker to craft a trusted Semrush link that silently redirects victims to any malicious destination.

# Vulnerable Endpoint
```
https://out.pwnedbox.site/?url=
```

# Steps to Reproduce
1. Open browser and navigate to:

```
   https://out.pwnedbox.site/?url=https://bing.com
```

2. Observe the request originates from trusted `out.pwnedbox.site` domain
3. The `url` parameter is processed with zero domain validation
4. Browser is immediately redirected to `bing.com`

# Payload Used
```
?url=https://bing.com
```

# Proof of Concept
<img width="955" height="515" alt="a1" src="https://github.com/user-attachments/assets/f7a960d0-c76f-4afe-9f90-f99e165e3578" />
<img width="955" height="515" alt="a2" src="https://github.com/user-attachments/assets/00139a35-89fb-422c-b30d-35e0457e80b0" />
<img width="958" height="513" alt="a3" src="https://github.com/user-attachments/assets/30624b61-31ed-46b6-a0ce-9a1a8d97b8c0" />

# Impact
- Phishing via trusted Semrush brand targeting SEO/marketing professionals

# Recommendation for fix
- Allowlist only `semrush.com` and owned domains in the `url` parameter

