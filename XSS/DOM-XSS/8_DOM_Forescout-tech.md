# Title
DOM-based Cross-Site Scripting (XSS) via URL Hash Fragment in Fancybox Handler

# Vulnerability Type
DOM-XSS

# Summary
The application is vulnerable to DOM-based XSS through its URL hash fragment routing. When the page loads, a script reads the `window.location.hash`, URL-decodes it, and attempts to match it against existing Fancybox anchors. If no matching anchor is found, the script falls back to injecting the decoded hash directly into an HTML container (`#inline-preview`) using jQuery's unsafe `.html()` method. Because the input is not sanitized or HTML-encoded, an attacker can craft a URL with a malicious payload in the hash fragment, resulting in arbitrary JavaScript execution.

**Vulnerable Code Analysis (Source to Sink):**

- **Source (Tainted Data):** The application reads the raw URL hash fragment.
```js
var hash = window.location.hash; // e.g., #payload
var decoded = decodeURIComponent(hash.substring(1));
```

- **Flawed Processing:** The script attempts to match the hash to a known anchor. If it fails (which it will for a custom payload), it routes the data to the vulnerable fallback.
```js
var $anchor = $('a.fancybox-inline[href="' + hash + '"]:first');
if ($anchor.length) {
    // Normal path
} else {
    $('#inline-preview').html(decoded); // Sink
}
```

- **Sink (Execution Point):** The decoded user input is injected into the DOM using `.html()`, which parses the input as HTML and executes any embedded event handlers or scripts.
```js
$('#inline-preview').html(decoded);
```

# Vulnerable Endpoint
`https://kzlabs.store/subdomains/gallery/`

# Steps to Reproduce
1. Navigate to the vulnerable gallery page: `https://kzlabs.store/subdomains/gallery/`
2. Append the malicious payload to the URL hash fragment.
3. Load the following crafted URL:  
`https://kzlabs.store/subdomains/gallery/#rix4uni'%22%3C%3E%3CiMg%20src=x%20onerror=confirm(1)%3E`
4. Upon page load, the script will fail to find a matching anchor, pass the decoded payload into the `.html()` sink, and execute the `confirm(1)` alert box.

# Payload Used
**URL Hash Payload:**
```
#rix4uni'%22%3C%3E%3CiMg%20src=x%20onerror=confirm(1)%3E
```
# Proof of Concept
<img width="1915" height="1034" alt="1" src="https://github.com/user-attachments/assets/67f89d7b-54cd-4250-bf9d-68c3e3afbe9b" />
<img width="1915" height="623" alt="2" src="https://github.com/user-attachments/assets/0ecb3935-d6bb-4463-911f-72d88efe3796" />
<img width="1920" height="1041" alt="3" src="https://github.com/user-attachments/assets/146ae02a-6655-4085-a666-c4edcf0657d2" />

**Resulting DOM structure (when injected):**
```html
<div class="inline-preview-bar" id="inline-preview">
    rix4uni'"><><iMg src=x onerror=confirm(1)>
</div>
```
# Impact
- **Arbitrary JavaScript Execution:** An attacker can execute malicious scripts in the context of the victim's browser session by tricking them into clicking a crafted URL.
- **Account Takeover:** Attackers can steal session cookies or local storage tokens, allowing them to impersonate the victim.
- **Data Exfiltration:** Malicious scripts can read sensitive data (like user details, authentication tokens, or page content) and send it to an attacker-controlled server.
- **Phishing:** Attackers can modify the DOM to display fake login forms or overlays to steal credentials.

# Recommendation for fix
The vulnerability occurs because user-controllable data from the URL is inserted into the DOM using `.html()`.

1. **Use Safe Text Insertion:** If the fallback preview is meant to display plain text, use `.text()` instead of `.html()`. This prevents the browser from parsing the input as HTML.
2. **Avoid Dynamic Selectors:** Ensure that user input is not used to construct jQuery selectors directly.
3. **Sanitize if HTML is Required:** If HTML must be rendered in the preview (which is rarely needed), use a robust, context-aware sanitization library (like DOMPurify) to strip dangerous tags and attributes before passing it to `.html()`.

**Secure Code Example:**
```js
$(window).on('load', function() {
    var hash = window.location.hash;

    if (hash.length > 1) {
        var decoded = decodeURIComponent(hash.substring(1));
        
        // Use attribute selector safely
        var $anchor = $('a.fancybox-inline').filter(function() {
            return $(this).attr('href') === hash;
        }).first();

        if ($anchor.length) {
            $anchor.delay(700).trigger('click');
        } else {
            // SECURE FIX: Use .text() to safely render as plain text
            $('#inline-preview').text(decoded); 
        }
    }
});
```
