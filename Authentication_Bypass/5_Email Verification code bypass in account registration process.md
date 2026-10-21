Lab name : Email Verification code bypass in account registration process
# Title
Authentication Bypass via Client-Side Response Manipulation During Registration
# Summary
The application relies entirely on a client-side JSON response to determine if a user has successfully verified their email address during the registration process. When an incorrect verification code is submitted, the server correctly returns a failure status (`"success": false`). However, the frontend application trusts this response to proceed. By intercepting the server's rejection response and manipulating the boolean value from `false` to `true`, an attacker can bypass the email verification step entirely, creating a fully verified account without possessing a valid code sent to the email address.

# Vulnerable Endpoint 
- `https://hackers.pwnedbox.site/api/verify/confirm` (POST request)

# Vulnerable Parameter
- `JSON Response body parameter: `"success"`

# Steps to Reproduce

1. Initiate the registration process on the application (`https://hackers.pwnedbox.site`) and proceed to the final step: **"Step 4 of 4: Verify your email"**.
2. Enter an arbitrary 6-digit code (e.g., `111111`) that is incorrect.
3. Ensure Burp Suite is configured to intercept requests and responses.
4. Click the **"Verifying..."** button. Intercept the `POST` request to `/api/verify/confirm`. The request body will contain the email and the incorrect code:
```json
{
  "email": "hello@gmail.com",
  "code": "111111"
}
```
5. Forward the request. Intercept the **Response** from the server. Observe that the server correctly rejects the invalid code with a JSON response:
```json
{
  "success": false,
  "message": "The verification code you entered is incorrect or has expired."
}
```
6. Manipulate the response body. Change `"success": false` to `"success": true`:
```json
{
  "success": true,
}
```
7. Forward the manipulated response to the browser.
8. Observe that the frontend application accepts the manipulated response, bypasses the email verification step, and redirects the user to the authenticated user dashboard, displaying an **"Email verified"** badge .


# Payload Used 
Manipulation of the JSON response body:  
`"success": false` -> `"success": true`

# Proof of Concept
<img width="1920" height="1038" alt="1" src="https://github.com/user-attachments/assets/5f139ce6-789f-407e-8f7a-dfb7c74f2aa6" />
<img width="1899" height="1041" alt="2" src="https://github.com/user-attachments/assets/1d84fe99-92aa-40ee-aced-7f60725aff1a" />
<img width="1920" height="1039" alt="3" src="https://github.com/user-attachments/assets/3c249f65-0c03-47fd-8ce1-b47b0734e94a" />
<img width="1920" height="1039" alt="4" src="https://github.com/user-attachments/assets/f72f276b-da01-429b-80ef-604a3357d580" />

# Impact

This vulnerability allows an unauthenticated attacker to bypass the mandatory email verification process during account registration.

By manipulating the server's response in transit, an attacker can force the frontend application to believe the email was successfully verified. Successful exploitation allows an attacker to:

- **Bypass Email Verification:** Create and access fully verified accounts without proving ownership of the provided email address.
- **Account Creation with Fake Emails:** Register accounts using arbitrary, unowned, or non-existent email addresses, which can be used to abuse platform resources, conduct spam, or evade platform bans.
- **Defeat Onboarding Security:** Undermine the integrity of the registration process, allowing unauthorized access to features intended only for verified users (such as connecting with coaches or volunteers).

# Recommedation for fix
1. **Server-Side Verification Enforcement:** The most critical fix is to enforce the email verification status strictly on the server side. When a user successfully verifies their email, the server must update their status in the database (e.g., `is_verified = true`) and generate a valid session token or cookie. Access to the dashboard and verified-user features must be protected by server-side middleware that checks this database flag, not a client-side JSON response.
2. **Do Not Rely on Client-Side Flags:** The application should not use frontend JavaScript to read a boolean value like `"success": true` and then decide to grant access or redirect to the dashboard. The server should only return a successful HTTP status and set a session cookie if the verification code genuinely matches the one stored in the backend for that specific user.
3. **Rate Limiting:** Implement strict rate limiting on the `/api/verify/confirm` endpoint to prevent brute-force attacks against the 6-digit code.
