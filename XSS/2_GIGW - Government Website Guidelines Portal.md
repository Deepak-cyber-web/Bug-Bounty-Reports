Lab name : Reflected XSS - GIGW: Government Website Guidelines Portal

# Title
Reflected Cross-Site Scripting (XSS) via `s` parameter

# Vulnerability Type
Reflected XSS

# Summary
In https://kzlabs.store/subdomains/guidelines/index.php?s= is vulnerable to reflected XSS. Website reflects the User input from search parameter which leads attacker to add malicious javascript code within victims browser when user visit malicious URL .

# Vulnerable Endpoint
https://kzlabs.store/subdomains/guidelines/index.php?s=

# Steps to Reproduce
1. Navigate to the following URL : https://kzlabs.store/subdomains/guidelines/index.php?s=%3Cscript%3Ealert(1)%3C/script%3E
2. Observe that a JavaScript alert box pops up displaying `1` — confirming that the script executed.

# Payload Used
`<script>alert(1)</script>`

# Proof of Concept

<img width="960" height="505" alt="3" src="https://github.com/user-attachments/assets/1857e186-09b4-475c-a361-b98de760651c" />

<img width="960" height="510" alt="4" src="https://github.com/user-attachments/assets/2d303724-e89a-4ead-8b5f-72cb4b1476eb" />


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


