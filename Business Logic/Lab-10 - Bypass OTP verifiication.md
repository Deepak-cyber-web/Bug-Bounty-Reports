# Title
Authentication Bypass via Client-Side OTP Response Manipulation — Server Does Not Re-Validate Session Server-Side

# Vulnerability Type
Business Logic Flaw — Authentication Bypass via Response Tampering (Client Trusts Manipulated Server Response; No Server-Side OTP Enforcement)

# Summary
The Mattermost customer portal email verification flow sends the OTP token to the server via POST to `/api/v1/users/verify-email-complete`. The server returns a JSON response with `"ok":true/false` to indicate success or failure. The client-side application trusts this response value to decide whether to grant access — without any independent server-side session or re-validation. By submitting a **deliberately wrong OTP** (`111111`), intercepting the server's failure response in Burp Suite, and changing `"ok":false` to `"ok":true` (with valid account data in the body), the client accepted the manipulated response as a successful verification and granted full authenticated access to the Mattermost customer portal — **without ever knowing the correct OTP**.

# Vulnerable Endpoint
`POST https://mattermost.pwnedbox.site/api/v1/users/verify-email-complete
Content-Type: application/json`

# Steps to Reproduce
1. Navigate to the Mattermost customer portal registration/login flow.
2. Enter email `gekko1@gmail.com` — a verification email is sent with a 6-digit OTP (real code: `013221`).
3. On the **Verify your email** screen, enter a **wrong OTP**: `111111`.
4. Enable Burp Suite Intercept ON before submitting.
5. Intercept the POST request to `/api/v1/users/verify-email-complete`:
    
    ```json
    {"email": "gekko1@gmail.com", "token": "111111"}
    ```
    
6. Right-click the intercepted request → **Do intercept → Response to this request**.
7. Click Forward — intercept the server's response.
8. In the response body, change:
    ```json
    {"ok": false}
    ```
    to:
    ```json
    {"ok": true, "account": {"email": "gekko@gmail.com", "name": "gekko", "company": "gekko"}}
    ```
    
9. Forward the modified response.
10. The client reads `"ok":true`, accepts it as verified — **user is now logged in** to the portal as `gekko`.
11. Dashboard confirms: Subscriptions, Purchase History, Profile pages accessible — full account access.

# Payload Used
**Original server response (wrong OTP):**

```json
{"ok": false}
```

**Manipulated response (exploit):**

```json
{
  "ok": true,
  "account": {
    "email": "gekko@gmail.com",
    "name": "gekko",
    "company": "gekko"
  }
}
```

Tool used: Burp Intercept → **Do intercept → Response to this request** to intercept and modify the server response before the client reads it.

# Proof of Concept
<img width="1920" height="990" alt="a1" src="https://github.com/user-attachments/assets/c9375fce-1620-44c4-ab19-c8c9758bad7d" />
<img width="1920" height="1045" alt="a2" src="https://github.com/user-attachments/assets/7450e6eb-baaf-4b65-9ec4-4802e1f07b10" />
<img width="1910" height="1035" alt="a3" src="https://github.com/user-attachments/assets/7bc056e8-da68-4f13-9905-3d118c5942da" />
<img width="1920" height="1037" alt="a4" src="https://github.com/user-attachments/assets/49d6f166-3aa1-4c84-ac8b-b88950e1da51" />


# Impact
- Authentication Bypass
- Account Takeover
# Recommendation for fix
**Issue a signed, `HttpOnly`, server-side session token only upon successful OTP verification** — present it on every request, validate it server-side. Client-readable flags like `ok:true` can always be flipped; a server-validated token cannot.
