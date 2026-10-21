Lab name : Reflected XSS - BigBasket: Online Supermarket Catalog

# Title
Reflected Cross-Site Scripting (XSS) via `q` parameter

# Vulnerability Type
Reflected XSS

# Summary
In https://kzlabs.store/subdomains/bigbasket/index.php?q= is vulnerable to reflected XSS. Website reflects the User input from search parameter which leads attacker to add malicious javascript code within victims browser when user visit malicious URL .

# Vulnerable Endpoint
https://kzlabs.store/subdomains/bigbasket/index.php?q=

# Steps to Reproduce
1. Navigate to the following URL : https://kzlabs.store/subdomains/bigbasket/index.php?q=%3Cscript%3Ealert%281%29%3C%2Fscript%3E
2. Observe that a JavaScript alert box pops up displaying `1` — confirming that the script executed.

# Payload Used
`<script>alert(1)</script>`

# Proof of Concept

<img width="960" height="509" alt="9" src="https://github.com/user-attachments/assets/6ec6e3b6-5a96-4173-92f2-8d942b3b8926" />

<img width="960" height="510" alt="10" src="https://github.com/user-attachments/assets/a7758049-9033-4a24-97f9-366678f4a7a9" />


# Impact
An attacker can perform the following actions using this vulnerability:
- **Arbitrary JavaScript Execution:** An attacker can execute malicious scripts in a victim's browser via a crafted URL.
- **Session Hijacking:** Theft of session cookies (if `HttpOnly` is not set), leading to account takeover.
- **Phishing/Defacement:** Attackers can inject fake login forms or alter page content to steal credentials.
- **Client-Side Exploitation:** Redirection to malicious sites or performing unauthorized actions on behalf of the user.

# Recommendation for fix
1. **Output Encoding:** Apply context-aware HTML entity encoding to the `q` parameter before reflecting it in the response (e.g., convert `<` to `&lt;`).
2. **Input Validation:** Implement strict allow-list validation, rejecting input containing HTML tags or special characters.
3. **Content Security Policy (CSP):** Implement a strict CSP (e.g., `script-src 'self'`) to block inline script execution.
4. **Secure Cookies:** Ensure all session cookies use the `HttpOnly` and `Secure` flags.
