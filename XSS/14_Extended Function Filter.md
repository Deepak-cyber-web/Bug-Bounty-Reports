
Lab name : Reflected XSS -  Extended Function Filter

# Title
Reflected Cross-Site Scripting (XSS) via `lname` parameter

# Vulnerability Type
Reflected XSS

# Summary
In https://account.pwnedbox.site/?fname=&lname= is vulnerable to reflected XSS. Website reflects the User input from search parameter which leads attacker to add malicious javascript code within victims browser when user visit malicious URL .

# Vulnerable Endpoint
https://account.pwnedbox.site/?fname=&lname=

# Steps to Reproduce
1. Navigate to the following URL : https://account.pwnedbox.site/?fname=hello&lname=cyberbro%27%22%3C%3E%3CsCrIpT%3Ea%5Cu006cert%281%29%3C%2FsCrIpT%3E
2. Observe that a JavaScript alert box pops up displaying `1` — confirming that the script executed.

# Payload Used
`cyberbro'"<><sCrIpT>a\u006cert(1)</sCrIpT>`

# Proof of Concept

<img width="1920" height="1035" alt="24" src="https://github.com/user-attachments/assets/4e6bc707-5eed-4746-ba07-f182de4ce985" />

<img width="1920" height="1036" alt="25" src="https://github.com/user-attachments/assets/635c42a8-6ccb-4505-90e0-662985687795" />

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
