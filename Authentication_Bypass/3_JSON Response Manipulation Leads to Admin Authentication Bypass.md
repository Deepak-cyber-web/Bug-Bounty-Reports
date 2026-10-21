Lab name : JSON Response Manipulation Leads to Admin Authentication Bypass
# Title
Authentication Bypass via via Client-Side Response Manipulation in Administrator Login
# Vulnerability Type
Authentication Bypass
# Summary
The application relies entirely on a client-side JSON response to determine if a user has successfully authenticated and to assign their privileges. When invalid credentials are submitted, the server correctly returns a failure status (`"status": "0"`). However, the frontend application trusts this response to grant access. By intercepting the server's response and manipulating the `"status"` parameter to `"1"` and the `"user_type"` to `"admin"`, an attacker can completely bypass the authentication mechanism and gain unauthorized access to the Enterprise Management Console with administrative privileges.

# Vulnerable Endpoint 
- `https://hub.pwnedbox.site/index.php` (POST request)

# Vulnerable Parameter
- JSON Response body parameters: `"status"` and `"user_type"`

# Steps to Reproduce

1. Navigate to the ControlHub login page: `https://hub.pwnedbox.site`
2. Enter an arbitrary username (e.g., `admin`) and an **incorrect** password.
3. Ensure Burp Suite is configured to intercept requests and responses.
4. Click the **"Sign In"** button. Intercept the `POST` request to `/index.php`.
5. Forward the request. Intercept the **Response** from the server. Observe that the server correctly rejects the invalid credentials with a JSON response:
```json
 
{
    "status": "0",
    "user_type": ""
 }
```
6. Manipulate the response body. Change `"status": "0"` to `"status": "1"` and set `"user_type"` to `"admin"`:
```json
{
    "status": "1",
    "user_type": "admin"
}
```
6. Forward the manipulated response to the browser.
7. Observe that the frontend application accepts the manipulated response, bypasses the login screen, and redirects the user to the **Enterprise Management Console** dashboard, displaying full administrative access (as seen in the second screenshot).

# Payload Used 
Manipulation of the JSON response body:  
`"status": "0"` -> `"status": "1"`  
`"user_type": ""` -> `"user_type": "admin"`

# Proof of Concept
<img width="1920" height="1036" alt="6" src="https://github.com/user-attachments/assets/d84a3d01-43c5-4020-bc96-76c0a717abf1" />
<img width="1914" height="1037" alt="7" src="https://github.com/user-attachments/assets/6f8df945-5346-4ad7-a317-422dfb433608" />
<img width="1920" height="1037" alt="8" src="https://github.com/user-attachments/assets/b5c705af-e85a-453f-8d75-c43925bbfb40" />
<img width="1913" height="1037" alt="9" src="https://github.com/user-attachments/assets/dc0ec2a2-a937-4cc8-92ed-8ea18dc8ad1d" />

# Impact
This vulnerability allows an unauthenticated attacker to completely bypass the administrator login mechanism.

By manipulating the server's response in transit, an attacker can force the frontend application to believe they have successfully authenticated as an administrator. Successful exploitation allows an attacker to:

- **Gain Full Administrative Access:** Access the Enterprise Management Console without valid credentials.
- **Access Sensitive Data:** View internal system metrics, server IP addresses, database status, and a list of all users (including emails and roles).
- **Potential for Further Compromise:** While this specific bypass grants dashboard access, an attacker with administrative UI access may be able to perform privileged actions (such as modifying users, deleting data, or accessing internal APIs) depending on the backend's authorization checks.

# Recommedation for fix
1. **Server-Side Authentication Enforcement:** The most critical fix is to enforce authentication and authorization strictly on the server side. The frontend must **never** be the sole authority on whether a user is authenticated. The server must generate and issue a valid, cryptographically signed session token (or secure cookie) only after it has genuinely verified the credentials against the database.
2. **Do Not Rely on Client-Side Flags:** The application should not use frontend JavaScript to read a boolean value like `"status": "1"` and then decide to render the admin dashboard. The server should return an HTTP `302` redirect to the dashboard _only_ if the backend successfully validates the login and establishes a server-side session. Access to dashboard routes must be protected by server-side middleware that checks for this valid session.
3. **Role-Based Access Control (RBAC):** The server must independently verify the user's role (`user_type`) on every request to sensitive endpoints. Do not trust the role sent from the client or derived solely from a manipulated login response.
