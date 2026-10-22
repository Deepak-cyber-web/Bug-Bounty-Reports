# Title
Reflected Cross-Site Scripting (XSS) in Apache Tomcat Example Page (`numguess.jsp`)

**CVE:** CVE-2026-50229  
**Severity:** Medium

# Vulnerable Endpoint
`http://67.223.117.254/examples/jsp/num/numguess.jsp`

# Description
The Apache Tomcat example application endpoint `/examples/jsp/num/numguess.jsp` reflects the value of the `hint` query parameter directly into the HTTP response without any output encoding or sanitization. This allows an attacker to inject arbitrary JavaScript into the page, which executes in the context of the victim's browser when a crafted URL is visited.

The presence of the default Tomcat `examples` web application on an internet-facing host is itself a misconfiguration, as these sample applications are intended for demonstration only and are known to contain insecure coding patterns.

# Steps to Reproduce
1. Open a web browser.
2. Navigate to the following URL:
```text
http://67.223.117.254/examples/jsp/num/numguess.jsp?guess=5&hint=<script>alert(document.domain)</script>
```
(URL-encoded form:)
```text
http://67.223.117.254/examples/jsp/num/numguess.jsp?guess=5&hint=%3Cscript%3Ealert%28document.domain%29%3C%2Fscript%3E
```
1. Observe that a JavaScript alert dialog appears displaying the document domain (`67.223.117.254`), confirming that the injected script was executed.
2. Inspect the page source to confirm the `hint` parameter value is reflected unescaped in the HTML response.

# Proof of Concept
<img width="1918" height="1003" alt="1" src="https://github.com/user-attachments/assets/1295c914-8fa7-43cc-af30-121450ca1779" />

# Impact
An attacker can craft a malicious link that, when clicked by a victim, executes arbitrary JavaScript under the trusted origin `http://67.223.117.254/`. This enables:

- **Session hijacking** — theft of session cookies (where `HttpOnly` is not set), leading to account takeover.
- **Credential phishing** — DOM manipulation to display fake login forms on the legitimate domain.
- **Malicious redirection** — redirecting victims to attacker-controlled sites.
- **Forced actions** — performing state-changing requests on behalf of the authenticated victim.
- **Client-side defacement** — altering page content to damage trust.
Although exploitation requires user interaction, the reflected nature and exposed default application increase the likelihood of successful social-engineering attacks.

# Remediation
1. **Remove the `examples` webapp** from the production Tomcat deployment (delete the `webapps/examples` directory or disable it). This is the primary and recommended fix, as sample applications have no production purpose.
2. **Encode all reflected output** if the page must remain — use OWASP Java Encoder, JSTL `<c:out>`, or `fn:escapeXml()` to HTML-encode the `hint` parameter before rendering.
3. **Validate input** — restrict the `hint` parameter to an expected allowlist of characters/values.
4. **Deploy a Content-Security-Policy (CSP)** header to reduce the impact of any residual script injection (e.g., `default-src 'self'`).
5. **Harden cookies** — set `HttpOnly`, `Secure`, and `SameSite` attributes on session cookies.
6. **Add security headers** — `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`.
7. **Re-scan** the host with a full-coverage Nuclei run (all severities and broader template tags) to identify any additional exposed default applications or vulnerabilities.
