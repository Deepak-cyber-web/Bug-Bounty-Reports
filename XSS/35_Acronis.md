Lab name : Stored XSS - Acronis
# Title
Stored Cross-Site Scripting (XSS) via User Profile Signature
# Vulnerability Type
Stored Cross-Site Scripting (XSS)
# Summary
The application is vulnerable to Stored Cross-Site Scripting (XSS) through the user profile edit functionality. The application fails to properly sanitize or encode user-supplied input in the **"Signature"** field before storing it in the database and subsequently rendering it on the user's profile page and potentially alongside their forum posts. This allows an attacker to inject a malicious JavaScript payload that is permanently stored on the server. When any user (including administrators or other community members) views the infected profile or a thread containing the attacker's post, the malicious script executes automatically in their browser.

# Vulnerable Endpoint
- `https://forum.pwnedbox.site` (My Profile / Edit Profile Page)

# Vulnerable Parameter
- `Signature` (within the PROFILE SIGNATURE section)
# Steps to Reproduce

1. Log in to the Acronis Forum with a valid account.
2. Navigate to the profile editing page by clicking on **"My Profile"** in the left-hand navigation menu (`https://forum.pwnedbox.site`).
3. Scroll down to the **"PROFILE SIGNATURE"** section.
4. In the **"Signature"** text box, insert the following payload:  
    `cyberbro"><><iMg sRc=x OnErRor=confirm(100)>`
5. Click the **"Save Profile"** button.
6. Observe that a JavaScript alert box pops up displaying `100`, confirming the script executed.
7. Refresh the page or view the profile from a different account/browser. Observe that the alert executes again automatically, confirming the payload is permanently stored and executes for any user who views the profile (or posts containing the signature).

# Payload Used
`cyberbro"><><iMg sRc=x OnErRor=confirm(100)>`

# Proof of Concept
<img width="1920" height="1040" alt="21" src="https://github.com/user-attachments/assets/871279ed-e218-49ca-b4f9-4601907b0d84" />
<img width="1920" height="1039" alt="22" src="https://github.com/user-attachments/assets/c6825bda-96d5-4a0e-a57c-99a8f8b5f5e7" />

# Impact

An attacker can execute arbitrary JavaScript in the browser of any user who views the infected discussion thread. Because the payload is stored, it executes automatically without requiring the victim to click a link. This allows an attacker to:

- **Steal Session Cookies:** If cookies lack the `HttpOnly` flag, an attacker can steal session tokens and hijack user accounts, including administrators.
- **Perform Actions on Behalf of the Victim:** The script can make authenticated requests to change the victim's password, delete posts, or modify account settings.
- **Phishing:** Inject fake login forms into the page to steal credentials.
- **Defacement:** Modify the content of the discussion thread or redirect users to malicious external sites.

# Recommendation for fix

1. **Output Encoding:** Apply context-aware HTML entity encoding to all user-supplied comment data before rendering it in the browser (e.g., convert `<` to `&lt;`, `>` to `&gt;`, `"` to `&quot;`). This prevents the browser from interpreting input as HTML or JavaScript.
2. **Input Sanitization:** Use a reputable HTML sanitization library (such as DOMPurify) to strip dangerous tags and attributes from the comment input before storing it in the database. If rich text is not required, strip all HTML tags entirely.
3. **Content Security Policy (CSP):** Deploy a strict CSP header (e.g., `script-src 'self'`) to prevent the execution of inline scripts and unauthorized external scripts as a defense-in-depth measure.

