Lab name : Stored XSS - Video Streaming Service (Netflix Clone)
# Title
Stored Cross-Site Scripting (XSS) via Movie Viewer Reactions

# Vulnerability Type
Stored Cross-Site Scripting (XSS)

# Summary
The application is vulnerable to Stored Cross-Site Scripting (XSS) through the movie reaction functionality. The application fails to properly sanitize or encode user-supplied input in the **"Share Your Thoughts & Reactions"** text area before storing it in the database and subsequently rendering it on the **movie details page**. This allows an attacker to inject a malicious JavaScript payload that is permanently stored on the server. When any user views the movie page and the reaction section, the malicious script executes automatically in their browser.

# Vulnerable Endpoint
- `https://netflix.pwnedbox.site/101` (Movie Details page where the payload is submitted and executed)

# Vulnerable Parameter
- `Share Your Thoughts & Reactions`
# Steps to Reproduce

1. Log in to the application with a valid account.
2. Navigate to any movie details page, for example: `https://netflix.pwnedbox.site/101`
3. Scroll down to the **"How are you feeling about this movie?"** section.
4. Select a mood (e.g., Mind Blown) and a rating (e.g., 5 Stars).
5. In the **"Share Your Thoughts & Reactions"** text box, insert the following payload:  
    `cyberbro"><><iMg sRc=x OnErRor=confirm(100)>`
6. Click the **"Post Reaction & Feeling"** button to submit.
7. Observe that a JavaScript alert box pops up displaying `100`, confirming the script executed.
8. Refresh the page or view the movie from a different account/browser to confirm the payload is permanently stored in the "Viewer Reactions" section and executes automatically for any user who views it.

# Payload Used
`cyberbro"><><iMg sRc=x OnErRor=confirm(100)>`

# Proof of Concept
<img width="1920" height="1036" alt="15" src="https://github.com/user-attachments/assets/54f60970-42ec-452c-b0c1-1254e819303d" />
<img width="1920" height="1037" alt="16" src="https://github.com/user-attachments/assets/ecb2925d-2aba-44df-9909-2f169f62b15d" />

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
