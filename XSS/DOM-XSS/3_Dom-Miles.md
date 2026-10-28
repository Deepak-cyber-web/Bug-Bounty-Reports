# Title
DOM-based Cross-Site Scripting (XSS) via URL Hash Fragment and `eval()` Execution

# Vulnerability Type
DOM-based Cross-Site Scripting (XSS)

# Summary
The application is vulnerable to DOM-based XSS through its URL hash fragment handling. The application reads the raw URL hash (`window.location.hash`), URL-decodes it, and inserts it into the DOM using the unsafe `innerHTML` property. To make matters worse, the code then actively searches for any `<script>` tags within the injected content and executes them using the dangerous `eval()` function. An attacker can exploit this by crafting a malicious URL with a payload in the hash fragment, resulting in arbitrary JavaScript execution in the victim's browser.

**Vulnerable Code Analysis (Source to Sink):**

- **Source (Tainted Data):** The application reads the raw URL hash fragment.
```js
var hashData = window.location.hash.substring(1);
```

- **Sink (Execution Point 1):** The input is decoded and injected into the DOM via `.innerHTML`
```js
welcomeEl.innerHTML = 'Welcome ' + decodeURIComponent(hashData);
```

- **Sink (Execution Point 2):** The code explicitly extracts the text content of any injected `<script>` tags and passes it to `eval()`, forcing execution.
```js
var scripts = welcomeEl.getElementsByTagName('script');
for (var i = 0; i < scripts.length; i++) {
  eval(scripts[i].innerText || scripts[i].textContent);
}
```

# Vulnerable Endpoint
`https://kzlabs.store/subdomains/milesweb/`

# Steps to Reproduce
1. Navigate to the vulnerable MilesWeb page: `https://kzlabs.store/subdomains/milesweb/`
2. Append the malicious payload to the URL hash fragment.
3. Load the following crafted URL:  
    `https://kzlabs.store/subdomains/milesweb/#rix4uni"><><Script>confirm(1)</Script>`
4. Upon page load, the `checkHashXSS()` function runs, processes the hash, injects the payload, and uses `eval()` to execute the script.
5. Observe the `confirm(1)` alert box executing.

# Payload Used
**URL Hash Payload:**
```
#rix4uni"><><Script>confirm(1)</Script>
```
# Proof of Concept
<img width="1920" height="1044" alt="1" src="https://github.com/user-attachments/assets/f9034830-e497-46c0-a6ee-c67066149407" />
<img width="1920" height="1037" alt="2" src="https://github.com/user-attachments/assets/664b3305-91fa-4b73-87c7-dd3693360e50" />
<img width="1920" height="1040" alt="3" src="https://github.com/user-attachments/assets/b98efd72-b919-4c42-8cf0-90e7d9825710" />

**Resulting DOM structure:**
```html
<div id="welcome" class="welcome-banner mb-3" style="display: block; ...">
    Welcome rix4uni"><><script>confirm(1)</script>
</div>
```
# Impact
- **Arbitrary JavaScript Execution:** An attacker can execute malicious scripts in the context of the victim's browser session by tricking them into clicking a crafted link.
- **Bypass of Native Protections:** Because the code explicitly uses `eval()`, it bypasses the browser's built-in security mechanisms that normally prevent `<script>` tags injected via `innerHTML` from executing.
- **Account Takeover:** Attackers can steal session cookies or local storage tokens, allowing them to impersonate the victim.
- **Phishing:** Attackers can modify the DOM to display fake login forms or overlays to steal credentials.

# Recommendation for fix

The vulnerability occurs because user-controllable data from the URL is inserted into the DOM using `.innerHTML` and then executed via `eval()`.

1. **Remove `eval()`:** The `eval()` loop must be completely removed. If the application needs to render HTML, it should use a robust sanitization library (like DOMPurify). However, in this context, rendering raw HTML from the URL is unnecessary.
2. **Use Safe Text Insertion:** Use `.textContent` instead of `.innerHTML`. This treats the input as plain text and prevents the browser from parsing it as HTML.
3. **Validate/Encode the Input:** If the parameter is meant to display a name or a simple string, validate it against a strict allow-list (e.g., alphanumeric characters only) or HTML-encode it.

**Secure Code Example:**
```js
function checkHashXSS() {
  if (window.location.hash) {
    var hashData = window.location.hash.substring(1);
    if (hashData) {
      var welcomeEl = document.getElementById('welcome');
      if (welcomeEl) {
        welcomeEl.style.display = 'block';
        
        // SECURE FIX: Use textContent to prevent HTML parsing
        welcomeEl.textContent = 'Welcome ' + decodeURIComponent(hashData);
        
        // SECURE FIX: Remove the eval() loop entirely
      }
    }
  }
}
```
