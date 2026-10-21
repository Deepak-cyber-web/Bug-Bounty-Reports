Lab name: Reflected XSS - Path Based

# Title
Reflected Cross-Site Scripting (XSS) via https://kzlabs.store/subdomains/path-fetch/ path

# Vulnerability Type
Reflected XSS

# Summary
In https://kzlabs.store/subdomains/path-fetch/about is vulnerable to reflected XSS. Website reflects the User input from URL path which leads attacker to add malicious javascript code within victims browser when user visit malicious URL .

# Vulnerable Endpoint
https://kzlabs.store/subdomains/path-fetch/about

# Steps to Reproduce
1. Navigate to the following URL : https://kzlabs.store/subdomains/path-fetch/cyberbro'%22%3C%3E%3CsCriPT%3Econfirm(1)%3C/sCript%3E
2. Observe that a JavaScript alert box pops up displaying `1` — confirming that the script executed.

# Payload Used
`cyberbro'"<><sCriPT>confirm(1)</sCript>`

# Proof of Concept

<img width="960" height="509" alt="17" src="https://github.com/user-attachments/assets/d5e657c6-260d-4080-af19-44153123a62f" />

<img width="959" height="511" alt="18" src="https://github.com/user-attachments/assets/d0e1918b-db16-4215-a892-e8e83024bcfb" />


# Impact
An attacker can perform the following actions using this vulnerability:
- **Arbitrary JavaScript Execution:** An attacker can execute malicious scripts in a victim's browser via a crafted URL.
- **Session Hijacking:** Theft of session cookies (if `HttpOnly` is not set), leading to account takeover.
- **Phishing/Defacement:** Attackers can inject fake login forms or alter page content to steal credentials.
- **Client-Side Exploitation:** Redirection to malicious sites or performing unauthorized actions on behalf of the user.

# Recommendation for fix
1. **Avoid Reflecting Raw Paths:** Do not reflect raw, unvalidated path segments directly into error pages or headers (e.g., "Page Not Found: https://kzlabs.store/subdomains/path-fetch/hello"). Sanitize or URL-encode the path before display.
2. **Path Segment Validation:** Implement strict allow-list validation for URL path segments (e.g., usernames, slugs, or IDs). The application should only accept expected characters (like alphanumeric and hyphens) and reject or return a 404 error for paths containing HTML special characters (`<`, `>`, `"`, `'`).
3. **Output Encoding:** Apply context-aware HTML entity encoding to the `q` parameter before reflecting it in the response (e.g., convert `<` to `&lt;`).
4. **Input Validation:** Implement strict allow-list validation, rejecting input containing HTML tags or special characters.
5. **Content Security Policy (CSP):** Implement a strict CSP (e.g., `script-src 'self'`) to block inline script execution.
6. **Secure Cookies:** Ensure all session cookies use the `HttpOnly` and `Secure` flags.
