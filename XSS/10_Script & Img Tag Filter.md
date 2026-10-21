Lab name: Reflected XSS - Script & Img Tag Filter

# Title
Reflected Cross-Site Scripting (XSS) via `fname` and `lname` parameter

# Vulnerability Type
Reflected XSS

# Summary
In https://kzlabs.store/subdomains/support/?fname=&lname= is vulnerable to reflected XSS. Website reflects the User input from URL path which leads attacker to add malicious javascript code within victims browser when user visit malicious URL .

# Vulnerable Endpoint
https://kzlabs.store/subdomains/support/?fname=&lname=

# Steps to Reproduce
1. Navigate to the following URL : https://kzlabs.store/subdomains/support/?fname=cyberbro%27%22%3C%3E%3CiMg+sRc%3Dx+oneRror%3Dconfirm%281%29%3E&lname=cyberbro%27%22%3C%3E%3CiMg+sRc%3Dx+oneRror%3Dconfirm%281%29%3E
2. Observe that a JavaScript alert box pops up displaying `1` — confirming that the script executed.

# Payload Used
`cyberbro'"<><iMg sRc=x oneRror=confirm(1)>`

# Proof of Concept
<img width="1919" height="1029" alt="19" src="https://github.com/user-attachments/assets/91ec5df3-d949-4b45-902a-b6148d70f767" />

<img width="1920" height="1040" alt="20" src="https://github.com/user-attachments/assets/266c244b-0293-4aa8-8bfc-eccc1044e2dc" />


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
