# Title
DOM-based Cross-Site Scripting (XSS) via `document.write()` in Search Tracking Function

# Vulnerability Type
DOM-based Cross-Site Scripting (XSS)

# Summary
The application is vulnerable to DOM-based XSS through its client-side search functionality. When a user visits the page with a search query in the URL, the application extracts the `search` parameter and passes it to a tracking function (`trackSearch()`). This function uses `document.write()` to dynamically generate an `<img>` tag, concatenating the unsanitized user input directly into the HTML string. Because the input is not encoded, an attacker can break out of the `src` attribute and inject arbitrary HTML/JavaScript, resulting in script execution in the victim's browser.

**Vulnerable Code Analysis (Source to Sink):**

- **Source (Tainted Data):** The application extracts user input directly from the URL query string.
```js
var query = (new URLSearchParams(window.location.search)).get('search');
```

- **Flawed Processing:** The unsanitized variable is passed directly into a tracking function.
```js
if (query) {
    // ... UI updates ...
    trackSearch(query);
}
```

- **Sink (Execution Point):** The `trackSearch()` function uses `document.write()` to parse a string into the DOM. The unsanitized `query` variable is concatenated directly into the `src` attribute.
```js
function trackSearch(query) {
    document.write('<img src="/useruploads/resources/images/tracker.gif?searchTerms=' + query + '">');
}
```

# Vulnerable Endpoint
`https://kzlabs.store/subdomains/anilist/`

# Steps to Reproduce
1. Navigate to the vulnerable AniList subdomain: `https://kzlabs.store/subdomains/anilist/`
2. Append the malicious payload to the `search` parameter in the URL.
3. Load the following crafted URL:  
    `https://kzlabs.store/subdomains/anilist/?search=rix4uni%27%22%3C%3E%3Cscript%3Econfirm(1)%3C%2Fscript%3E`
4. Upon page load, the application reads the parameter, passes it to `trackSearch()`, and `document.write()` injects the payload into the DOM.
5. Observe the `confirm(1)` alert box executing.

# Payload Used
**URL Query Parameter Payload:**
```
?search=rix4uni%27%22%3C%3E%3Cscript%3Econfirm(1)%3C%2Fscript%3E
```
# Proof of Concept
<img width="1920" height="1037" alt="1" src="https://github.com/user-attachments/assets/6f2bdf42-ef3f-444a-875e-6bbed74f0ca6" />
<img width="1920" height="1043" alt="2" src="https://github.com/user-attachments/assets/d2280ab8-c505-46d1-b8e1-e29b790f8832" />
<img width="1920" height="1037" alt="3" src="https://github.com/user-attachments/assets/edc9b3d7-45d2-43aa-8ab0-a0d0a3e69649" />


**Resulting DOM structure:**
```js
<script>
    document.write('<img src="/useruploads/resources/images/tracker.gif?searchTerms=rix4uni'"><><script>confirm(1)</script>">');
</script>
```
# Impact
- **Arbitrary JavaScript Execution:** An attacker can execute malicious scripts in the context of the victim's browser session by tricking them into clicking a crafted link.
- **Account Takeover:** Attackers can steal session cookies or local storage tokens, allowing them to impersonate the victim.
- **Data Exfiltration:** Malicious scripts can read sensitive data (like user details, watchlists, or page content) and send it to an attacker-controlled server.
- **Phishing:** Attackers can modify the DOM to display fake login forms or overlays to steal credentials.

# Recommendation for fix
The vulnerability occurs because user-controllable data is inserted into the DOM using `document.write()` without proper encoding. `document.write()` is generally deprecated and highly dangerous when handling dynamic input.

1. **Avoid `document.write()`:** Replace `document.write()` with safe DOM creation APIs like `document.createElement()`.
2. **Safe Attribute Assignment:** Assign the user input to the `src` property of the image element directly. This ensures the browser treats the input as a string value and does not parse it as HTML.
3. **Encode URL Parameters:** Even when assigning via properties, URL parameters should be properly encoded using `encodeURIComponent()` to prevent breaking the request or injecting unintended query parameters.

**Secure Code Example:**
```js
function trackSearch(query) {
    // Create the image element safely
    var img = document.createElement('img');
    
    // Securely append the query to the URL using encodeURIComponent
    img.src = '/useruploads/resources/images/tracker.gif?searchTerms=' + encodeURIComponent(query);
    
    // Append to the body (or a specific container)
    document.body.appendChild(img);
}
```
