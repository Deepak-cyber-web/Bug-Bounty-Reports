Lab name : Stored XSS - Product Reviews (Flipkart Clone)
# Title
Stored Cross-Site Scripting (XSS) via Product Reviews Section

# Vulnerability Type
Stored Cross-Site Scripting (XSS)

# Summary
The application is vulnerable to Stored Cross-Site Scripting (XSS) through the Product Review functionality. The application fails to properly sanitize or encode user-supplied input in the comment text area before storing it in the database and subsequently rendering it on the discussion page. This allows an attacker to inject a malicious JavaScript payload that is permanently stored on the server. When any user (including administrators or other community members) views the affected discussion thread, the malicious script executes in their browser.

# Vulnerable Endpoint
https://members.pwnedbox.site/index.php?product_id=1

# Vulnerable Parameter
 Customer Review section 

# Steps to Reproduce
1. Navigate to the application and log in with a valid account (or register a new account if required).
2. Navigate to the product page, for example: `https://members.pwnedbox.site/index.php?product_id=1`
3. Scroll down to the **"Write a Customer Review"** section.
4. In the **"Headline / Short Title"** and **"Detailed Review Comments"** text boxes, insert the following payload:  
    `cyberbro"><><iMg sRc=x OnErRor=confirm(1)>`
5. Click the **"Submit Product Review"** button to submit the review.
6. Observe that a JavaScript alert box pops up displaying `1`, confirming the script executed.
7. Refresh the page or view the product page from a different account/browser to confirm the payload is permanently stored and executes for any user who views the review.

# Payload Used
`cyberbro"><><iMg sRc=x OnErRor=confirm(100)>`

# Proof of Concept
<img width="1920" height="1040" alt="5" src="https://github.com/user-attachments/assets/656e8026-94ce-49bb-b966-c5c844ae02dc" />
<img width="1920" height="1041" alt="6" src="https://github.com/user-attachments/assets/f2002017-6b36-4b6a-a3f9-5a556f34163b" />

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
