Lab name : Stored XSS - Events & Polls Activity Center
# Title
Stored Cross-Site Scripting (XSS) via Events & Polls Activity Center

# Vulnerability Type
Stored Cross-Site Scripting (XSS)

# Summary
The application is vulnerable to Stored Cross-Site Scripting (XSS) through the Events & Polls Activity Center functionality. The application fails to properly sanitize or encode user-supplied input in the comment text area before storing it in the database and subsequently rendering it on the discussion page. This allows an attacker to inject a malicious JavaScript payload that is permanently stored on the server. When any user (including administrators or other community members) views the affected discussion thread, the malicious script executes in their browser.

# Vulnerable Endpoint
- `https://articles.pwnedbox.site/index.php` (Community Events tab)
- `https://articles.pwnedbox.site/index.php` (Interactive Polls tab)  
(Note: The vulnerability occurs on the authenticated dashboard page where these forms are rendered.)

# Vulnerable Parameter
- **Community Events:** `Event Title`, `Event Date & Time`, `Event Description`
- **Interactive Polls:** `Poll Question`, `Poll Description / Context`, `Poll Options`

# Steps to Reproduce

## **Part A: Stored XSS via Community Events**
1. Log in to the application with a valid account.
2. Navigate to the Events & Polls portal: `https://articles.pwnedbox.site/index.php`
3. Ensure you are on the **"Community Events"** tab.
4. In the **"Create Community Event"** form, insert the following payload into the **"Event Title"**, **"Event Date & Time"**, and **"Event Description"** fields:  
    `cyberbro"><><iMg sRc=x OnErRor=confirm(1)>`
5. Click the **"Publish Event"** button to submit the event.
6. Observe that a JavaScript alert box pops up displaying `1` (as seen in Screenshot 3), confirming the script executed.
7. Refresh the page or view the portal from a different account/browser to confirm the payload is permanently stored and executes automatically for any user who views the "Scheduled Events" list.

## Part B: Stored XSS via Interactive Polls
1. While still on the Events & Polls portal, click on the **"Interactive Polls"** tab.
2. In the **"Create Interactive Poll"** form, insert the payload into the **"Poll Question"**, **"Poll Description / Context"**, and the **"Poll Options"** fields:  
    `cyberbro"><><iMg sRc=x OnErRor=confirm(100)>`
3. Click the **"Publish Poll"** button to submit the poll.
4. Observe that a JavaScript alert box pops up displaying `100` (as seen in Screenshot 1), confirming the script executed.
5. Refresh the page to confirm the payload is permanently stored and executes for any user who views the "Active Polls" list.

# Payload Used
`cyberbro"><><iMg sRc=x OnErRor=confirm(1)>`
`cyberbro"><><iMg sRc=x OnErRor=confirm(100)>`

# Proof of Concept
<img width="1920" height="1037" alt="7" src="https://github.com/user-attachments/assets/c3918ad0-87fc-4427-9f50-2d829aff1164" />
<img width="1920" height="1042" alt="8" src="https://github.com/user-attachments/assets/7f5db92b-f900-450f-a2f7-89e97ba30499" />
<img width="1920" height="1041" alt="9" src="https://github.com/user-attachments/assets/4adf54e3-6e15-4aa6-8182-0065017cfc88" />
<img width="1920" height="1041" alt="10" src="https://github.com/user-attachments/assets/f638645e-979a-435d-bf77-bcb3bb008a0c" />

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
