Lab name : Stored XSS - Shopify
# Title
Stored Cross-Site Scripting (XSS) via Article HTML Source Editor

# Vulnerability Type
Stored Cross-Site Scripting (XSS)

# Summary
The application is vulnerable to Stored Cross-Site Scripting (XSS) through the article creation functionality. While the standard rich-text editor may attempt to sanitize input, the application provides a direct **HTML source editor** toggle. The application fails to sanitize or encode user-supplied HTML in this source view before saving it to the database and rendering it on the "My Articles" page. This allows an attacker to inject a malicious JavaScript payload that is permanently stored and executes automatically whenever a user views the articles list.

# Vulnerable Endpoint
- `https://cms.pwnedbox.site` (Article Creation/Editing and "My Articles" Listing Page)

# Vulnerable Parameter
- `Description` (specifically when edited via the `</>` HTML Source toggle)
# Steps to Reproduce

1. Log in to the application with a valid account.
2. Click on the **"Write"** button to create a new article.
3. In the **"Title"** field, insert the following payload:  
    `cyberbro"><><iMg sRc=x OnErRor=confirm(100)>`
4. In the **"Description"** field, click the **`</>` (HTML source)** toggle button. Insert the same payload:  
    `cyberbro"><><iMg sRc=x OnErRor=confirm(100)>`
5. Click the **"Save"** button to publish the article.
6. Observe that a JavaScript alert box pops up displaying `100`, confirming the script executed.
7. Navigate back to the **"My Articles"** page (`https://cms.pwnedbox.site`). Observe that the payload is permanently stored in the article card, and the alert executes automatically every time the page is loaded.

# Payload Used
`cyberbro"><><iMg sRc=x OnErRor=confirm(100)>`

# Proof of Concept
<img width="1920" height="1039" alt="19" src="https://github.com/user-attachments/assets/147358c1-8e89-4ee1-866a-1052e5697349" />

<img width="1920" height="1037" alt="20" src="https://github.com/user-attachments/assets/8369b747-80c3-42b1-8058-063b7c2eb24c" />

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
