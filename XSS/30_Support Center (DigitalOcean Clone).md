Lab name : Stored XSS - Support Center (DigitalOcean Clone)
# Title
Stored Cross-Site Scripting (XSS) via Support Center

# Vulnerability Type
Stored Cross-Site Scripting (XSS)

# Summary
The application is vulnerable to Stored Cross-Site Scripting (XSS) through the Support Center functionality. The application fails to properly sanitize or encode user-supplied input in the **ticket creation form** before storing it in the database and subsequently rendering it on the **tickets page**. This allows an attacker to inject a malicious JavaScript payload that is permanently stored on the server. When any user (including administrators or other support staff) views the affected **ticket**, the malicious script executes in their browser.

# Vulnerable Endpoint
- `https://digitalocean.pwnedbox.site/index.php?view=create` (Where the payload is submitted)
- `https://digitalocean.pwnedbox.site/index.php?view=tickets` (Where the payload executes upon submission and listing)

# Vulnerable Parameter
- `Subject`
- `Description`
# Steps to Reproduce

1. Log in to the application with a valid account.
2. Navigate to the Support Center page: `https://digitalocean.pwnedbox.site/index.php?view=create`
3. Scroll down to the **"Create Support Ticket"** form.
4. In the **"Subject"** and **"Description"** fields, insert the following payload:  
    `cyberbro"><><iMg sRc=x OnErRor=confirm(100)>`
5. Click the **"Submit Support Case"** button.
6. Observe that a JavaScript alert box pops up displaying `100` immediately upon redirection to the Tickets list (`https://digitalocean.pwnedbox.site/index.php?view=tickets`), confirming the script executed.
7. Click on the newly created ticket to view it. Observe that the alert executes again, confirming the payload is permanently stored and executes for any user (including administrators) who views the ticket.

# Payload Used
`cyberbro"><><iMg sRc=x OnErRor=confirm(100)>`

# Proof of Concept

<img width="1920" height="1037" alt="11" src="https://github.com/user-attachments/assets/2f1f2800-2363-4c18-a375-d4e3263cd3b0" />
<img width="1920" height="1038" alt="12" src="https://github.com/user-attachments/assets/4550be81-a52d-48e0-91fd-53ab8a621a13" />

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
