# Title
Self-Cross-Site Scripting (Self-XSS) in Search Parameter

# Vulnerability Type
Self-XSS

# Summary
A Self-XSS vulnerability exists in the search functionality of the KrazePlanet application. The application reflects unsanitized user-supplied input via the `search` POST parameter directly into the HTML response. Because the input is not encoded, an attacker can execute arbitrary JavaScript in their own browser. While this requires the user to manually input the payload (hence "Self-XSS"), it violates security best practices and can be leveraged by attackers using social engineering tactics (e.g., "Copy/Paste this code into the search bar to win a prize") to execute malicious actions on the victim's behalf.

# Vulnerable Endpoint
`https://forms.pwnedbox.site`

# Steps to Reproduce
1. Navigate to the target application: `https://forms.pwnedbox.site`.
2. Locate the main search bar on the page.
3. Enter the following payload into the search field: `"><script>confirm(1)</script>`
4. Click the **Search** button.
5. Observe that a JavaScript `confirm` dialog box pops up displaying "1", indicating successful script execution.

# Payload Used
```html
rix4uni'"<><script>confirm(1)</script>
```

# Proof of Concept
<img width="1920" height="1039" alt="a5" src="https://github.com/user-attachments/assets/681ffe4a-a8a8-4097-bfad-afba7e369dcd" />
<img width="1920" height="1041" alt="a4" src="https://github.com/user-attachments/assets/257eef7c-6fd6-49ea-bacd-a4f240236acc" />

# Impact
- **Social Engineering Vector:** While Self-XSS cannot be triggered directly by an attacker via a crafted link (like Reflected XSS), an attacker can trick a victim into pasting a malicious payload into the search bar. This could lead to actions being performed on the victim's behalf, credential theft via fake login prompts injected into the DOM, or session hijacking if the payload is designed to exfiltrate cookies.
- **DOM Manipulation:** An attacker can use the injected script to alter the appearance of the web page, potentially defacing it or displaying misleading information to the user.

# Recommendation for fix
- **Output Encoding:** Ensure all user-supplied data is properly HTML-encoded before being rendered in the browser. Convert characters like `<`, `>`, `"`, and `'` to their respective HTML entities. Use the framework's native encoding functions (e.g., `htmlspecialchars($input, ENT_QUOTES, 'UTF-8')` in PHP) to neutralize the payload.
- **Input Validation:** Implement strict server-side input validation on the search parameter, restricting characters to an acceptable whitelist (e.g., alphanumeric and spaces) and rejecting requests containing HTML tags or script characters.
- **Security Headers (CSP):** Implement a strict Content Security Policy (CSP) to mitigate the impact of any XSS vulnerabilities. For example, `Content-Security-Policy: script-src 'self';` will block inline scripts like the one used in the payload.
