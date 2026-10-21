Lab name: Reflected XSS - Imgur Mobile

# Title
Reflected Cross-Site Scripting (XSS) via https://kzlabs.store/subdomains/media/index.php/account/SaturnV/messages path

# Vulnerability Type
Reflected XSS

# Summary
In [https://kzlabs.store/subdomains/media/index.php/account/SaturnV/messages](https://kzlabs.store/subdomains/media/index.php/account/SaturnV/messages) is vulnerable to reflected XSS. The website reflects the User input from the URL path, which leads the attacker to add malicious JavaScript code within the victim's browser when the user visits a malicious URL. The `SaturnV/messages` path folder location is vulnerable to reflected XSS.

# Vulnerable Endpoint
https://kzlabs.store/subdomains/media/index.php/account/SaturnV/messages

# Steps to Reproduce
1. Navigate to the following URL : https://kzlabs.store/subdomains/media/index.php/account/cyberbro'%22%3C%3E%3CsCriPT%3Econfirm(1)%3C/sCript%3E/messages
2. Observe that a JavaScript alert box pops up displaying `1` — confirming that the script executed.

# Payload Used
`cyberbro'"<><sCriPT>confirm(1)</sCript>`

# Proof of Concept
<img width="1917" height="1035" alt="43" src="https://github.com/user-attachments/assets/bb7c441d-1014-4417-8589-426d7d88946c" />
<img width="1920" height="1031" alt="44" src="https://github.com/user-attachments/assets/49c6c4ad-9f79-4b61-a3d8-664e8e2a5a25" />

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
