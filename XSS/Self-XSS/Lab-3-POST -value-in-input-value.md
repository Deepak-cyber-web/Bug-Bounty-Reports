# Title
Self-Cross-Site Scripting (Self-XSS) in POST-Input Tag Value

# Vulnerability Type
Self-XSS

# Summary
A Self-XSS vulnerability exists in the "What's your name?" form on the application. The application takes user-supplied input from the `name` POST parameter and reflects it back into the HTML response without proper sanitization or encoding. An attacker can exploit this by tricking a victim into manually pasting a malicious JavaScript payload into the input field. Once submitted, the script executes in the context of the victim's browser.

# Vulnerable Endpoint
`https://survey.pwnedbox.site/`
# Steps to Reproduce
1. Navigate to the target application: `https://survey.pwnedbox.site/`.
2. Locate the input field labeled "Name:".
3. Enter the following payload into the input field: `rix4uni""><sCript>confirm(1)</scriPt>`
4. Click the **Enter** button to submit the form.
5. Observe that a JavaScript `confirm` dialog box pops up displaying the number "1", indicating successful script execution.

# Payload Used
```html
rix4uni'"<><script>confirm(1)</script>
```

# Proof of Concept
<img width="1920" height="1035" alt="a1" src="https://github.com/user-attachments/assets/7a48e49e-0f46-4410-9c77-01940fbb7b45" />
<img width="1920" height="1040" alt="a2" src="https://github.com/user-attachments/assets/07960550-22eb-49fb-879e-5377f0516987" />

# Impact
- **Social Engineering Vector:** While Self-XSS cannot be triggered directly by an attacker via a crafted link (like Reflected XSS), an attacker can trick a victim into pasting a malicious payload into the search bar. This could lead to actions being performed on the victim's behalf, credential theft via fake login prompts injected into the DOM, or session hijacking if the payload is designed to exfiltrate cookies.
- **DOM Manipulation:** An attacker can use the injected script to alter the appearance of the web page, potentially defacing it or displaying misleading information to the user.
  
# Recommendation for fix
- **Output Encoding:** Ensure all user-supplied data is properly HTML-encoded before being rendered in the browser. Convert characters like `<`, `>`, `"`, and `'` to their respective HTML entities. Use the framework's native encoding functions (e.g., `htmlspecialchars($input, ENT_QUOTES, 'UTF-8')` in PHP) to neutralize the payload.
- **Input Validation:** Implement strict server-side input validation on the search parameter, restricting characters to an acceptable whitelist (e.g., alphanumeric and spaces) and rejecting requests containing HTML tags or script characters.
- **Security Headers (CSP):** Implement a strict Content Security Policy (CSP) to mitigate the impact of any XSS vulnerabilities. For example, `Content-Security-Policy: script-src 'self';` will block inline scripts like the one used in the payload.
