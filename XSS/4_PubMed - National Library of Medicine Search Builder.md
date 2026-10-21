Lab Name : Reflected XSS - PubMed: National Library of Medicine Search Builder
# Title
Reflected Cross-Site Scripting (XSS) via `term` parameter

# Vulnerability Type
Reflected XSS

# Summary
In https://kzlabs.store/subdomains/pubmed/index.php?term= is vulnerable to reflected XSS. Website reflects the User input from search parameter which leads attacker to add malicious javascript code within victims browser when user visit malicious URL .

# Vulnerable Endpoint
https://kzlabs.store/subdomains/pubmed/index.php?term=

# Steps to Reproduce
1. Navigate to the following URL : https://kzlabs.store/subdomains/pubmed/index.php?term=%3Cscript%3Ealert%281%29%3C%2Fscript%3E
2. Observe that a JavaScript alert box pops up displaying `1` — confirming that the script executed.

# Payload Used
`<script>alert(1)</script>`

# Proof of Concept
<img width="960" height="509" alt="7" src="https://github.com/user-attachments/assets/1901328b-0daa-44c3-a77b-24ea38f193b1" />

<img width="960" height="509" alt="8" src="https://github.com/user-attachments/assets/51f1fa60-e2a4-485e-a541-89b533479844" />

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


