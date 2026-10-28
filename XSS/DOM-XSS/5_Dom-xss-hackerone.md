Lab name : DOM XSS -  Hackerone

# Title
DOM-based Cross-Site Scripting (XSS) via `lever-` URL Parameter

# Vulnerability Type
DOM-XSS

# Summary
The application is vulnerable to DOM-based XSS on the careers page. The JavaScript code reads the URL query string (`?lever-`), decodes it, and inserts it directly into the DOM without sanitization, allowing an attacker to execute arbitrary JavaScript.

**Vulnerable Code Analysis (Source to Sink):**

- **Source (Tainted Data):** The application reads attacker-controlled input from the URL.

```js
var pageUrl = window.location.href;
var pageUrlSplit = pageUrl.split('?lever-');
leverParam = '?lever-' + decodeURIComponent(pageUrlSplit[1]); // Decodes encoded payloads
```

- **Sink (Execution Point):** The tainted `leverParam` is concatenated into an HTML string and passed to `jQuery.append()`, which parses it as HTML.

```js
var link = hostedUrl + leverParam; // Tainted string
jQuery('#jobs-list').append(
    '<a class="job-title" href="' + link + '">' + job.title + '</a>' // Sink
);
```

# Vulnerable Endpoint
`https://kzlabs.store/subdomains/careers/`

# Steps to Reproduce
1. Navigate to the vulnerable careers page: `https://kzlabs.store/subdomains/careers/`
2. Append the malicious payload to the `?lever-` parameter in the URL:  
    `https://kzlabs.store/subdomains/careers/?lever-securitycyberbro%27%22%3C%3E%3CsCript%3Econfirm(1)%3C/sCript%3E`
3. Press Enter to load the URL.
4. Observe that a JavaScript `confirm(1)` alert box pops up, demonstrating successful script execution.
5. Inspect the page (F12) to see the payload reflected and executed inside the `<a class="job-title">` element.

# Payload Used
`?lever-securitycyberbro%27"<><sCript>confirm(1)</sCript>`

# Proof of Concept
<img width="1920" height="1037" alt="1" src="https://github.com/user-attachments/assets/070a0496-d08d-4644-90f2-43e11fa5eef5" />
**Screenshot 1:** The successful execution of the payload. The URL bar shows the injected payload, and the `confirm(1)` alert box proves script execution.
<img width="1919" height="627" alt="2" src="https://github.com/user-attachments/assets/b3dcc33a-4f08-4f61-ab48-a301d546fbb9" />
**Screenshot 2:** The DOM inspection showing the exact injection point. The payload breaks out of the `href` attribute and creates a new `<script>` tag.

**Resulting DOM structure:**
```HTML
<a class="job-title" href="/a1b2c3d4-senior-security-engineer?lever-securitycyberbro'"><><sCript>confirm(1)</sCript>">
    "Senior Security Engineer"
</a>
```
# Impact
- **Account Takeover:** Attackers can steal session cookies/tokens to impersonate the victim.
- **Data Theft & Phishing:** Injected scripts can exfiltrate sensitive data or display fake login forms to steal credentials.
- **Trust Exploitation:** Because the payload executes on a legitimate domain, victims are highly likely to click the crafted link.

# Recommendation for fix
- **Use an Allow-list (Input Validation):** Since the `?lever-` parameter is only used for filtering teams, strictly validate it against a hardcoded list of expected values (e.g., `engineering`, `security`, `sales`). Reject everything else.
- **Avoid HTML String Concatenation (Safe Sinks):** Do not pass user-controlled data into `jQuery.append('<a href="'+link+'">')`. Instead, create DOM elements programmatically using jQuery's safe methods: `.attr('href', link)` to safely set attributes, and `.text(job.title)` to safely set text content.
- **Remove `decodeURIComponent()`:** If input is properly validated and safely inserted into the DOM, there is no need to manually decode URL parameters in the client-side script.
- Context-Aware Output Encoding (If concatenation must be used): If the code must construct HTML strings, the leverParam must be HTML-entity encoded (and URL-encoded) before being placed inside the href attribute. This ensures characters like ", ', <, and > are converted to &quot;, &#x27;, &lt;, and &gt;, preventing the attacker from breaking out of the attribute.
