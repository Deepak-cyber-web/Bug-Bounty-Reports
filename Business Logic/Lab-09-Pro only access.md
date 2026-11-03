# Title
Free Plan Users Can Assign Pro-Only "View Access" (Read) Role via `access_level` Parameter Tampering in Magic Code Invite Request

# Vulnerability Type
Business Logic Flaw — Insufficient Server-Side Plan Enforcement on Role Assignment; Subscription Tier Restriction Bypassable via Parameter Tampering

# Summary
Lovable restricts the "View Access" (read-only) invite role to Pro Plan subscribers. Free plan users can only generate invites with "Edit Access" (write). However, the `access_level` parameter sent in the POST request to `/projects/{project-id}/magic-codes` is not validated server-side against the user's subscription tier. By intercepting the request and changing `access_level` from `write` to `read`, a free plan user can generate a read-access invite link — a feature gated behind the Pro Plan — without paying. The server accepts and reflects the `read` value in the response without any plan check, and the generated link grants the invited user Viewer permission confirmed inside the project's People section.

# Vulnerable Endpoint
```
POST https://api.lovable.dev/projects/{project-id}/magic-codes
Authorization: Bearer <token>
Content-Type: application/json
```

# Steps to Reproduce
1. Create two accounts — **Account A** (owner, Free plan) and **Account B** (invitee).
2. From Account A, create a new project (choose any template, click Remix).
3. Open the project and click **Share** (top right corner).
4. Observe: the UI shows "View access" is disabled — **Pro Plan required** to enable read-only invites.
5. In the Share panel, set invite link access to **Edit access** to ensure the endpoint fires.
6. Enable Burp Suite Intercept ON.
7. Toggle the access to trigger the POST request to `/projects/{id}/magic-codes`.
8. Intercept the POST request — request body:
    
    ```json
    {"access_level": "write"}
    ```
    
9. Change `"access_level": "write"` to `"access_level": "read"` and forward the request.
10. (Optional) Intercept the response — `read` is reflected back with no error.
11. Refresh the project's Share panel — invite link now shows **View access** despite being on the Free plan.
12. Copy the invite link, paste it in Account B, and accept the invitation.
13. Account A → Share → People you invited — Account B shows **Viewer** permission confirmed.

# Payload Used
**Original request body (Free plan default):**
```json
{"access_level": "write"}
```

**Tampered request body (exploit):**
```json
{"access_level": "read"}
```

**Full request:**

```http
POST /projects/{project-id}/magic-codes HTTP/2
Host: api.lovable.dev
Authorization: Bearer <token>
Content-Type: application/json

{"access_level": "read"}
```

Change: `access_level` value changed from `write` to `read`. Server accepts the value without checking the user's subscription plan.

# Proof of Concept
<img width="1920" height="1035" alt="a1" src="https://github.com/user-attachments/assets/d3811ae9-f091-4a81-9053-3866129317ff" />
<img width="1920" height="1036" alt="a2" src="https://github.com/user-attachments/assets/0d740e41-b772-427a-9247-3afe388a9f3b" />
<img width="1920" height="1033" alt="a3" src="https://github.com/user-attachments/assets/c3a72f3e-35f0-451f-86db-89df2b4dfb30" />
<img width="1920" height="1032" alt="a4" src="https://github.com/user-attachments/assets/e247a3d4-23d4-4a6e-9ef3-8aa82e7dff7a" />

# Impact
- Financial Impact

# Recommendation for fix
- **Enforce subscription plan server-side on every role assignment request.** Before accepting `access_level=read`, the API must query the user's current plan from the database and reject the request with HTTP 403 if they are not on Pro.
- Never rely on UI-level gating (disabled buttons, hidden options) to enforce paid feature access — the API endpoint must independently validate entitlement.
