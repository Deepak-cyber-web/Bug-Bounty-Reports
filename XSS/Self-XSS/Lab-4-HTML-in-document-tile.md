# Title
Self-Cross-Site Scripting (Self-XSS) via HTML Injection in Document Title

# Vulnerability Type
Self-XSS

# Summary
A Self-XSS vulnerability exists in the "What's your name?" form. The application reflects unsanitized user input into the `<title>` tag of the HTML document. Because the application fails to encode special characters, an attacker can break out of the `<title>` context and inject malicious HTML/JavaScript. While this requires the user to manually input the payload (Self-XSS), it can be leveraged via social engineering to execute arbitrary scripts in the victim's browser.

# Vulnerable Endpoint
`https://reports.pwnedbox.site`

# Steps to Reproduce
1. Navigate to the target application: `https://reports.pwnedbox.site`.
2. Locate the input field labeled "Name:".
3. Enter the following payload into the input field: `rix4uni""></title><iMg sRc=x onError=confirm(1)>`
4. Click the **Enter** button to submit the form.
5. Observe the page reload. The payload breaks out of the title tag, and the browser executes the JavaScript, resulting in a `confirm` dialog box displaying "1".

# Payload Used
```html
rix4uni""></title><iMg sRc=x onError=confirm(1)>
```

# Proof of Concept
<img width="1920" height="1034" alt="a2" src="https://github.com/user-attachments/assets/5d642639-743e-4d67-b51c-e7c416be7b4c" />
<img width="1920" height="1040" alt="a1" src="https://github.com/user-attachments/assets/2345e3b0-0e9d-49a7-8e69-28c5f41c7132" />


# Impact
- **Social Engineering Vector:** While Self-XSS cannot be triggered directly by an attacker via a crafted link (like Reflected XSS), an attacker can trick a victim into pasting a malicious payload into the search bar. This could lead to actions being performed on the victim's behalf, credential theft via fake login prompts injected into the DOM, or session hijacking if the payload is designed to exfiltrate cookies.
- **DOM Manipulation:** An attacker can use the injected script to alter the appearance of the web page, potentially defacing it or displaying misleading information to the user.
# Recommendation for fix
- **Output Encoding:** Ensure all user-supplied data is properly HTML-encoded before being rendered in the browser. Convert characters like `<`, `>`, `"`, and `'` to their respective HTML entities. Use the framework's native encoding functions (e.g., `htmlspecialchars($input, ENT_QUOTES, 'UTF-8')` in PHP) to neutralize the payload.
- **Input Validation:** Implement strict server-side input validation on the search parameter, restricting characters to an acceptable whitelist (e.g., alphanumeric and spaces) and rejecting requests containing HTML tags or script characters.
- **Security Headers (CSP):** Implement a strict Content Security Policy (CSP) to mitigate the impact of any XSS vulnerabilities. For example, `Content-Security-Policy: script-src 'self';` will block inline scripts like the one used in the payload.
