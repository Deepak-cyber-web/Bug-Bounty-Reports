# Title
Open Redirect via Unvalidated `url` Parameter on `/redirect` Endpoint

# Vulnerability Type
Open Redirect

# Summary
The `/redirect` endpoint on `hackevents.pwnedbox.site` accepts a user-controlled `url` parameter without validating the destination domain. Any user clicking a link from the trusted HackerOne Events domain can be silently redirected to an arbitrary external website.

# Vulnerable Endpoint
`https://hackevents.pwnedbox.site/redirect?url=`

# Steps to Reproduce
1. Open browser and navigate to:

```
   https://hackevents.pwnedbox.site/redirect?url=https://bing.com
```

2. Observe the request originates from the trusted `hackevents.pwnedbox.site` domain
3. Browser is redirected to external domain `bing.com` without any warning

# Payload Used
```
https://hackevents.pwnedbox.site/redirect?url=https://bing.com
```

# Proof of Concept
<img width="956" height="515" alt="a1" src="https://github.com/user-attachments/assets/f499aa9c-9394-4211-8832-671880be1ae1" />
<img width="957" height="513" alt="a2" src="https://github.com/user-attachments/assets/e1f75417-24b1-4bcf-9fe7-75647bf5e1fe" />
<img width="947" height="508" alt="a3" src="https://github.com/user-attachments/assets/4ed6db0b-c244-4572-aa2b-be17cb2c00de" />
<img width="954" height="515" alt="a4" src="https://github.com/user-attachments/assets/696cf716-da38-4acd-94e3-204215ed00c5" />

# Impact
- Phishing via trusted HackerOne brand — high-value target for security community
- 
# Recommendation for fix
- Allowlist only internal domains in the `url` parameter
- Reject any value with external `://` or non-whitelisted hostnames
- Replace open redirect with server-side route mapping (index → path)

