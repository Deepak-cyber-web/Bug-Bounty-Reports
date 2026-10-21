# Title
Authentication Bypass via Client-Side Email Verification Response Manipulation

# Vulnerability Type
Authentication Bypass

# Summary
The application relies on client-side validation to determine if an email verification code (OTP) is correct. When a user submits the OTP, the server responds with a JSON object indicating the verification status (`"verify": false`). However, the frontend application trusts this response to grant access to the dashboard. By intercepting the server's response and manipulating the `"verify"` parameter from `false` to `true`, an attacker can bypass the email verification process entirely and access authenticated areas of the application without possessing a valid OTP.

# Vulnerable Endpoint 
- `https://verify.pwnedbox.site/?action=verify` (POST request)

# Vulnerable Parameter
- JSON Response body parameter: `"verify"` (and potentially `"success_fraction"`)

# Steps to Reproduce
1. Register a new account on the application (`https://verify.pwnedbox.site`). The application will prompt for an email verification code (OTP).
2. Enter an **incorrect** OTP (e.g., `111111`) in the input fields and click "Verifying...".
3. Ensure Burp Suite is configured to intercept requests and responses.
4. In Burp Suite, intercept the `POST` request to `/?action=verify`. The request body will contain the email and the incorrect OTP:
```json
{
  "email": "hello@gmail.com",
  "otp": "111111"
}
```
5. Forward the request. Intercept the **Response** from the server. Observe that the server correctly rejects the invalid OTP with a JSON response:
```json
{
  "verify": false,
  "message": "Invalid OTP. Please try again."
}
```
6. Manipulate the response body. Change `"verify": false` to `"verify": true`. _(Optional: You may also need to change `"success_fraction": 0.0` to `1.0` depending on the frontend logic).
```json
{
  "verify": true,
}
```
7. Forward the manipulated response to the browser.
8. Observe that the frontend application accepts the manipulated response, bypasses the OTP requirement, and redirects the user to the authenticated dashboard (as seen in Screenshot 1), successfully bypassing the authentication control.

# Payload Used 
Manipulation of the JSON response body:  
`"verify": false` -> `"verify": true`

# Proof of Concept
<img width="1920" height="1039" alt="23" src="https://github.com/user-attachments/assets/14345875-d0a6-46d8-95a3-be5ef42a1965" />
<img width="1920" height="1080" alt="24" src="https://github.com/user-attachments/assets/7f37283d-1de4-44df-9355-4e22c578cdef" />
<img width="1920" height="1039" alt="26" src="https://github.com/user-attachments/assets/16d68f52-8499-4ef4-b233-1b7e10f272a2" />
<img width="1920" height="1033" alt="25" src="https://github.com/user-attachments/assets/d4ac054a-70e4-4c51-a38e-3873f2164d19" />
<img width="1903" height="1028" alt="27" src="https://github.com/user-attachments/assets/530aadec-9f0c-41cd-be93-0f0e0a5e1ac6" />

# Impact
This vulnerability allows an unauthenticated attacker to bypass the mandatory email verification (OTP) step during the registration process.

Because the frontend application relies on the server's JSON response (`"verify": true/false`) to determine if the OTP was correct, an attacker can intercept the server's rejection response for an invalid OTP and manipulate the boolean value to `true`.

Successful exploitation allows an attacker to:

- **Bypass Email Verification:** Gain access to the authenticated user dashboard without possessing a valid OTP or proving ownership of the email address.
- **Create Unverified Accounts:** Register and access accounts using fake, temporary, or unowned email addresses.
- **Undermine Onboarding Security:** Defeat the core logic of the registration process, allowing unauthorized access to features intended only for verified users.

# Recommedation for fix
1. **Server-Side Enforcement:** The most critical fix is to enforce access control on the server side. The server must **never** rely on the client to determine if a user is authenticated. When the user attempts to access the dashboard, the server must check the session state (e.g., a session cookie or token) that is only generated and marked as "verified" on the backend after a successful OTP validation.
2. **State Management:** When the OTP is successfully verified, the server should update the user's status in the database (e.g., `is_verified = true`) and generate an authenticated session token. Access to the dashboard must require this valid, server-side session, regardless of what the frontend JavaScript displays.
3. **Ignore Client-Side Trust:** Do not send sensitive status flags like `"verify": true/false` in a way that the client uses to grant access. The redirect to the dashboard should only happen if the server issues a `302` redirect or sets a valid session cookie upon a _genuinely_ correct OTP.
