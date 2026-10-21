Lab name : Admin Authentication Bypass Lead to Admin Account Takeover
# Title
Authentication Bypass via Client-Side Response Manipulation Leading to Admin Account Takeover
# Vulnerability Type
Authentication Bypass
# Summary
The application relies entirely on a client-side JSON response to determine if a user has successfully authenticated. When invalid credentials are submitted to the standard login form, the server correctly returns a failure status (`"status": false`). However, the frontend application trusts this response to grant access to the application. By intercepting the server's rejection response and manipulating the boolean value from `false` to `true`, an attacker can completely bypass the authentication mechanism and gain unauthorized access to the backend Admin Panel, resulting in full administrative privileges.

# Vulnerable Endpoint 
- `https://adminpanel.pwnedbox.site/?action=login` (POST request)

# Vulnerable Parameter
- JSON Response body parameter: `"status"`

# Steps to Reproduce

1. Navigate to the login page: `https://adminpanel.pwnedbox.site`
2. Enter an arbitrary username (e.g., `hello`) and an **incorrect** password.
3. Ensure Burp Suite is configured to intercept requests and responses.
4. Click the **"Sign In"** button. Intercept the `POST` request to `/?action=login`.
5. Forward the request. Intercept the **Response** from the server. Observe that the server correctly rejects the invalid credentials with a JSON response:
```json
{
  "status": false,
  "errorMessage": "Username and Password does not match."
}
```
6. Manipulate the response body. Change `"status": false` to `"status": true`:
```json
{
  "status": true,
}
```
7. Forward the manipulated response to the browser.
8. Observe that the frontend application accepts the manipulated response, bypasses the login screen, and redirects the user to the **Admin Panel** dashboard, granting full administrative access (as seen in the third screenshot).

# Payload Used 
Manipulation of the JSON response body:  
`"status": false` -> `"status": true`

# Proof of Concept
<img width="1920" height="1039" alt="1" src="https://github.com/user-attachments/assets/b3dffd92-fad7-47e0-8844-6c8920800c4f" />
<img width="1920" height="1038" alt="4" src="https://github.com/user-attachments/assets/e558644a-2542-4a25-a80d-02d15a1a249d" />
<img width="1906" height="1042" alt="2" src="https://github.com/user-attachments/assets/a6583a5e-898d-4783-88a8-adb124acef65" />
<img width="1916" height="1040" alt="3" src="https://github.com/user-attachments/assets/aa643297-5732-4018-b94e-c93603a7e9c2" />

# Impact

This vulnerability allows an unauthenticated attacker to bypass the standard login mechanism and achieve full Admin Account Takeover.

By manipulating the server's response in transit, an attacker can force the frontend application to believe they have successfully authenticated. Successful exploitation allows an attacker to:

- **Gain Unauthorized Admin Access:** Access the backend Admin Panel and all privileged functionalities without valid credentials.
- **Exposure of Sensitive PII:** View highly sensitive Personally Identifiable Information (PII) including customer names, physical addresses, tracking numbers, and shipment statuses.
- **Perform Privileged Actions:** Depending on the backend's authorization checks, the attacker may be able to perform administrative actions such as deleting shipment records (via the "Delete" buttons shown), modifying user accounts, or altering system settings.

# Recommedation for fix
1. **Server-Side Authentication Enforcement:** The most critical fix is to enforce authentication strictly on the server side. The frontend must **never** be the sole authority on whether a user is authenticated. The server must generate and issue a valid, cryptographically signed session token (or secure cookie) only after it has genuinely verified the credentials against the database.
2. **Do Not Rely on Client-Side Flags:** The application should not use frontend JavaScript to read a boolean value like `"status": true` and then decide to render the admin dashboard. The server should return an HTTP `302` redirect to the dashboard _only_ if the backend successfully validates the login and establishes a server-side session. Access to admin routes must be protected by server-side middleware that checks for this valid session.
3. **Role-Based Access Control (RBAC):** The server must independently verify the user's role on every request to sensitive endpoints. Do not trust the role sent from the client or derived solely from a manipulated login response.
