# Title
DOM-based Cross-Site Scripting (XSS) via URL Hash Fragment Router

# Vulnerability Type
DOM-XSS

# Summary
The application is vulnerable to DOM-based XSS within its Single Page Application (SPA) routing logic. The application reads the raw URL hash fragment (`window.location.hash`) to determine the active view. If the hash matches a valid route (e.g., `#send-transaction`), any additional characters appended after the route name are extracted, URL-decoded, and injected directly into the DOM using the unsafe `innerHTML` sink without proper sanitization. This allows an attacker to execute arbitrary JavaScript in the victim's browser.

**Vulnerable Code Analysis (Source to Sink):**

- **Source (Tainted Data):** Reading the raw hash fragment from the URL.

```js
var hash = window.location.hash.substring(1); // strips leading #
```

- **Processing/Flawed Logic:** Extracting the remainder of the string after a valid route and applying `decodeURIComponent()`.

```js
if (hash.indexOf(route) === 0) {
    // ...
    statusExtra = decodeURIComponent(hash.substring(route.length)); 
}
```

- **Sink (Execution Point):** Concatenating the tainted `statusExtra` variable into an HTML string and assigning it to `.innerHTML`.
```js
document.getElementById('conn-strip').innerHTML =
    '<span class="conn-ok">✓ Connected successfully</span>' +
    '<span class="conn-provider"> · ' + statusExtra + '</span>'; // Unsafe Sink
```
# Vulnerable Endpoint
`https://kzlabs.store/subdomains/wallet/`

# Steps to Reproduce
1. Navigate to the vulnerable application: `https://kzlabs.store/subdomains/wallet/`
2. Append a valid route (e.g., `#send-transaction`) followed by the malicious payload to the URL.
3. Ensure the payload is URL-encoded to pass through the application's `decodeURIComponent()` function.
4. The final crafted URL should look like this:  
    `https://kzlabs.store/subdomains/wallet/#send-transactioncyberbro'%22%3C%3E%3Cimg%20src=x%20onerror=confirm(1)%3E`
5. Press Enter to load the URL.
6. Observe that the `confirm(1)` alert box executes.
7. Inspect the DOM in the connection status strip (`.conn-strip`) to see the payload injected as raw HTML.

# Payload Used
```
#send-transactioncyberbro'"<><img%20src=x%20onerror=confirm(1)>
```

# Proof of Concept


# Impact
- **Arbitrary JavaScript Execution:** An attacker can execute malicious scripts in the context of the victim's browser session by tricking them into clicking a crafted link.
- **Account Takeover:** Attackers can steal session tokens, local storage data, or cookies, allowing them to impersonate the victim.
- **Data Exfiltration:** Malicious scripts can read sensitive data (like wallet addresses, balances, or transaction history) displayed in the application and send it to an attacker-controlled server.
- **Phishing:** Attackers can modify the DOM to display fake login prompts or transaction approval windows to steal credentials or trick users into signing malicious transactions.

# Recommendation for fix
The vulnerability occurs because user-controllable data from the URL is inserted into the DOM using `.innerHTML`.

1. **Avoid `innerHTML`:** Never pass unsanitized user input into `.innerHTML`. Instead, use `.textContent` to safely insert data, which treats the input as plain text rather than parsing it as HTML.
2. **Safe DOM Manipulation:** Use `document.createElement()` and `appendChild()` to build the DOM structure safely.
