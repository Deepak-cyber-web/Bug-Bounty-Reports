Lab name : Phone OTP Bypass via Response Manipulation
# Title
Authentication Bypass via via Client-Side SMS OTP Response Manipulation

# Vulnerability Type
Authentication Bypass

# Summary
The application relies on a client-side JSON response to determine if a user has successfully passed the SMS OTP (One-Time Password) verification step. When an incorrect OTP is submitted, the server responds with `"verify": false`. However, the frontend application trusts this response to grant access. By intercepting the server's response and manipulating the `"verify"` parameter from `false` to `true`, an attacker can bypass the phone number verification process entirely and gain access to the authenticated user dashboard without possessing a valid OTP.

# Vulnerable Endpoint 
- `https://mobile.pwnedbox.site/?action=verify` (POST request)

# Vulnerable Parameter
- JSON Response body parameter: `"verify"`

# Steps to Reproduce

1. Navigate to the application (`https://mobile.pwnedbox.site`) and initiate the login/registration process with a phone number.
2. When prompted for the OTP, enter an **incorrect** 6-digit code (e.g., `111111`).
3. Ensure Burp Suite is configured to intercept requests and responses.
4. In Burp Suite, intercept the `POST` request to `/?action=verify`. The request body will contain the phone number and the incorrect OTP:
```json
{
  "phone": "+911234567890",
  "otp": "111111"
}
```
5. Forward the request. Intercept the **Response** from the server. Observe that the server correctly rejects the invalid OTP with a JSON response:
```json
{
  "verify": false,
  "message": "Incorrect OTP. Please try again."
}
```
6. Manipulate the response body. Change `"verify": false` to `"verify": true`.
```json
{
  "verify": true,
}
```
7. Forward the manipulated response to the browser.
8. Observe that the frontend application accepts the manipulated response, bypasses the OTP requirement, and redirects the user to the authenticated dashboard, displaying "Number Verified" (as seen in the final screenshot).

# Payload Used 
Manipulation of the JSON response body:  
`"verify": false` -> `"verify": true`

# Proof of Concept
<img width="1918" height="1040" alt="1" src="https://github.com/user-attachments/assets/e0acf6db-fa71-45eb-8506-4d97bfaab932" />
<img width="1920" height="1044" alt="2" src="https://github.com/user-attachments/assets/22eb9a14-d49b-4545-962c-8aefb3f931dd" />
<img width="1910" height="1032" alt="3" src="https://github.com/user-attachments/assets/ab106f24-6d89-4eb5-952e-10177a2d33b0" />
<img width="1920" height="1039" alt="5" src="https://github.com/user-attachments/assets/a80e879b-857f-4be3-a47b-85c925b1e3a6" />
<img width="1912" height="1035" alt="4" src="https://github.com/user-attachments/assets/ffdd13f5-e923-42b8-b7ae-b9524bfe9c1c" />


# Impact
This vulnerability allows an unauthenticated attacker to bypass the mandatory SMS phone verification step.

By manipulating the server's response in transit, an attacker can force the frontend application to believe the OTP was correct. Successful exploitation allows an attacker to:

- **Bypass Phone Verification:** Gain access to the authenticated user dashboard and associated features without proving ownership of the phone number.
- **Account Creation/Login with Fake Numbers:** Register or log in to accounts using arbitrary or unowned phone numbers.
- **Defeat Onboarding Security:** Undermine the integrity of the authentication flow, allowing unauthorized access to areas intended only for verified users.

# Recommedation for fix
1. **Server-Side Access Control:** The most critical fix is to enforce access control strictly on the server side. The frontend must **never** be the sole authority on whether a user is authenticated. The server must generate and issue a valid session token (or set a secure session cookie) only after it has genuinely verified the OTP in its backend database.
2. **Do Not Rely on Client-Side Flags:** The application should not use frontend JavaScript to read a boolean value like `"verify": true` and then decide to render the dashboard. The server should return an HTTP `302` redirect to the dashboard _only_ if the backend successfully validates the OTP and establishes a server-side session. Access to dashboard routes must be protected by server-side middleware that checks for this valid session.
