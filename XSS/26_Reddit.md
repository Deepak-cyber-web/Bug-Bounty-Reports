Lab name: Reflected XSS - Reddit

# Title
Reflected Cross-Site Scripting (XSS) via https://kzlabs.store/subdomains/widgets/index.php/svc/shreddit/api/comments/askreddit/t3_u9po1l/t1_i5t2mnb path

# Vulnerability Type
Reflected XSS

# Summary
In https://kzlabs.store/subdomains/widgets/index.php/svc/shreddit/api/comments/askreddit/t3_u9po1l/t1_i5t2mnb is vulnerable to reflected XSS. The website reflects the User input from the URL path, which leads the attacker to add malicious JavaScript code within the victim's browser when the user visits a malicious URL. The `/t3_u9po1l/t1_i5t2mnb` path folder location is vulnerable to reflected XSS.

# Vulnerable Endpoint
https://kzlabs.store/subdomains/widgets/index.php/svc/shreddit/api/comments/askreddit/t3_u9po1l/t1_i5t2mnb
# Steps to Reproduce
1. Navigate to the following URL : https://kzlabs.store/subdomains/widgets/index.php/svc/shreddit/api/comments/askreddit/cyberbro'%22%3C%3E%3CiMg%20sRc=x%20oneRror=confirm(1)%3E/t1_i5u8kpl
2. Observe that a JavaScript alert box pops up displaying `1` — confirming that the script executed.

# Payload Used
`cyberbro'"<><iMg sRc=x oneRror=confirm(1)>`

# Proof of Concept
<img width="1920" height="1036" alt="45" src="https://github.com/user-attachments/assets/21bb68eb-7408-4899-aca0-981a117a599b" />
<img width="1920" height="1031" alt="46" src="https://github.com/user-attachments/assets/4f9d5a81-0395-4521-80ef-d7880926180f" />

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
