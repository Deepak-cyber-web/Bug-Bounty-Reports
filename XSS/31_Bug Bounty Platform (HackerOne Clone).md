Lab name : Stored XSS - Bug Bounty Platform (HackerOne Clone)
# Title
Stored Cross-Site Scripting (XSS) via Vulnerability Report Submission

# Vulnerability Type
Stored Cross-Site Scripting (XSS)

# Summary
The application is vulnerable to Stored Cross-Site Scripting (XSS) through the vulnerability report submission functionality. The application fails to properly sanitize or encode user-supplied input in the **report submission form** (Title, Description, and Impact fields) before storing it in the database and subsequently rendering it on the **report details page**. This allows an attacker to inject a malicious JavaScript payload that is permanently stored on the server. When any user (including triage teams, program managers, or administrators) views the affected **report**, the malicious script executes in their browser.

# Vulnerable Endpoint
- `https://hackerone.pwnedbox.site/index.php?view=submit` (Where the payload is submitted)
- `https://hackerone.pwnedbox.site/index.php?view=dashboard&report_id=5` (Where the payload executes upon viewing the report)

# Vulnerable Parameter
- `Title`
- `Description`
- 'Impact'
# Steps to Reproduce

1. Log in to the Bug Bounty Platform with a valid researcher account.
2. Navigate to the report submission page: `https://hackerone.pwnedbox.site/index.php?view=submit`
3. In the **"Title"**, **"Description"**, and **"Impact"** fields, insert the following payload:  
    `cyberbro"><><iMg sRc=x OnErRor=confirm(100)>`
4. Click the **"Submit Report"** button.
5. Observe that a JavaScript alert box pops up displaying `100` immediately upon redirection to the report view page, confirming the script executed.
6. Refresh the page, or view the report from a different account (e.g., an administrator or triage account). Observe that the alert executes again, confirming the payload is permanently stored and executes for any user who views the report.

# Payload Used
`cyberbro"><><iMg sRc=x OnErRor=confirm(100)>`

# Proof of Concept
<img width="1920" height="1039" alt="13" src="https://github.com/user-attachments/assets/7a10fa9c-955b-4a46-a07a-9ab9e045598e" />
<img width="1920" height="1039" alt="14" src="https://github.com/user-attachments/assets/6038547c-bed5-4ff5-9f32-d2da19c6274e" />

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
