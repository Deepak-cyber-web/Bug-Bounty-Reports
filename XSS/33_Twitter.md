Lab name : Stored XSS - Twitter
# Title
Stored Cross-Site Scripting (XSS) via Network Report Name

# Vulnerability Type
Stored Cross-Site Scripting (XSS)

# Summary
The application is vulnerable to Stored Cross-Site Scripting (XSS) through the Network Reports functionality. The application fails to properly sanitize or encode user-supplied input in the **"Report Name"** field when creating a new report. This allows an attacker to inject a malicious JavaScript payload that is permanently stored in the database and subsequently rendered unsafely in the **"Network Reports"** dashboard table. When any authenticated user (as the page states, "visible to every authenticated user") visits the reports page, the malicious script executes automatically in their browser.

# Vulnerable Endpoint
- `https://ads.pwnedbox.site` (Network Reports Dashboard)

# Vulnerable Parameter
- `Report Name` (within the "New Network Report" modal)
# Steps to Reproduce

1. Log in to the AdPulse application with a valid account.
2. On the Network Reports dashboard (`https://ads.pwnedbox.site`), click the **"+ New Network Report"** button.
3. In the **"Report Name"** field, insert the following payload:  
    `cyberbro"><><iMg sRc=x OnErRor=confirm(100)>`
4. Fill out the remaining fields (Ad Network, Date Range, Est. Impressions, Est. Clicks) with any valid values.
5. Click the **"Run & Save Report"** button.
6. Observe that a JavaScript alert box pops up displaying `100`, confirming the script executed.
7. Refresh the page or log in from a different account. Notice that the payload is permanently stored in the "Report Name" column of the table, and the alert executes automatically every time the page is loaded.

# Payload Used
`cyberbro"><><iMg sRc=x OnErRor=confirm(100)>`

# Proof of Concept
<img width="1920" height="1036" alt="17" src="https://github.com/user-attachments/assets/6eab0bd2-bdc4-4154-92b5-c522a3a25215" />
<img width="1920" height="1042" alt="18" src="https://github.com/user-attachments/assets/8b37df63-1e21-40b4-ba23-8b1034c5394c" />

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
