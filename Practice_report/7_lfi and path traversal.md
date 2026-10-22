# Title
Path Traversal and Local File Inclusion (CVE-2021-41773)

# Vulnerable Endpoint
`https://13.53.190.194/`

# Description
The target web server is vulnerable to a critical Path Traversal and Local File Inclusion (LFI) vulnerability, tracked as CVE-2021-41773. This flaw exists in Apache HTTP Server 2.4.49, caused by a path normalization bug. An attacker can bypass access controls and use directory traversal sequences (`../`) to access files outside the web root. In this instance, an attacker can manipulate the URL with encoded traversal payloads to read arbitrary files on the underlying operating system, such as the `/etc/passwd` file, which contains local user account information.

# Steps to Reproduce
1. Open a web browser or use a command-line tool like `curl`.
2. Craft a URL utilizing URL-encoded directory traversal sequences to target sensitive system files. The specific payload bypasses the Apache normalization process:  
    `https://13.53.190.194/icons/.%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/etc/passwd`
3. Request the URL.
4. The server responds with a `200 OK` status and outputs the contents of the `/etc/passwd` file.

# Proof of Concept
<img width="1914" height="745" alt="7" src="https://github.com/user-attachments/assets/79386b83-3d36-465f-ada3-0b2616174b58" />
<img width="1920" height="1037" alt="8" src="https://github.com/user-attachments/assets/eefee050-1282-4210-9de1-1e32471e243a" />


# Impact
- **Confidentiality Breach:** An unauthenticated attacker can read sensitive system files. Beyond `/etc/passwd`, attackers could potentially read application configuration files, private SSH keys, source code, or credentials hardcoded in scripts.
- **System Enumeration:** Extracting the `/etc/passwd` file provides a list of valid local usernames and reveals the presence of specific services running on the host (e.g., Jenkins, Cassandra, Docker). This information is highly valuable for subsequent privilege escalation or lateral movement.
- **Remote Code Execution (Potential):** If the server is configured with `mod_cgi` enabled (which is common in some legacy or misconfigured environments), this LFI can be escalated to Remote Code Execution (CVE-2021-42013). An attacker could execute arbitrary commands on the server.

# Remediation
1. **Immediate Patching:** Upgrade the Apache HTTP Server immediately to version 2.4.51 or later. Versions 2.4.49 and 2.4.50 are vulnerable to this and related bypass vulnerabilities.
2. **Temporary Mitigation:** If upgrading is not immediately possible, disable the `mod_cgi` module if it is not strictly required. Furthermore, you can add a rule in the Apache configuration to block requests containing URL-encoded directory traversal characters (`%2e`) or reject requests that resolve outside the document root.
3. **Least Privilege:** Ensure the Apache web server process is running under a low-privileged user account (e.g., `www-data` or `apache`) to limit the scope of files that can be read or modified, preventing access to highly sensitive files.
4. **Web Application Firewall (WAF):** Deploy or update a WAF to detect and block path traversal attack patterns.
