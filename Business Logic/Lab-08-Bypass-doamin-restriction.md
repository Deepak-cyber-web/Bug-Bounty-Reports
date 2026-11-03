# Title
Organization Domain Restriction Bypass via Null Byte (`%00`) Injection in Email Invite Field

# Vulnerability Type
Business Logic Flaw — Inconsistent Input Handling; Domain Blocklist Check and Email Delivery Parse the Address Differently (Null Byte Injection)

# Summary
Frontegg's invite system enforces an organization-level domain blocklist that blocks invitations to `yopmail.com`. The check that validates the domain and the system that actually delivers the invite email parse the email address differently when a URL-encoded null byte (`%00`) is injected. By submitting the email as `test@yopmail.com%00.evil.com`, the domain validation logic reads the domain as `evil.com` (not on the blocklist — allowed), while the email delivery system truncates the address at the null byte and delivers the invite to `test@yopmail.com` (the blocked domain). The invite was successfully delivered to the yopmail.com inbox, bypassing the organization's security restriction entirely.

# Vulnerable Endpoint
`POST https://frontegg.pwnedbox.site/index.php?action=invite`

# Steps to Reproduce
1. Log in to Frontegg as `gekko` in the `acme` organization.
2. Navigate to Invite Users (`/index.php?action=invite`).
3. Note the policy: **"Your organization's policy blocks invites to: yopmail.com"**.
4. Confirm the block works — try inviting `test@yopmail.com` directly:
    - Result: **"Blocked: test@yopmail.com — Sorry, you can't invite users with this domain."**
5. Now type the following crafted email address in the invite field:
    
    ```
    test@yopmail.com%00.evil.com
    ```
    
6. Click **Send invites**.
7. Result: **"Invite sent to test@yopmail.com%00.evil.com"** — no block error.
8. Check the yopmail.com inbox for `test@yopmail.com`:
    - **Invite email received** — subject: "Welcome to your Frontegg account!"
    - Activation link delivered to the blocked domain's inbox.

# Payload Used
**Blocked (normal input):**
```
test@yopmail.com
```

**Bypass payload:**
```
test@yopmail.com%00.evil.com
```

Mechanism: `%00` is a URL-encoded null byte. The domain blocklist check parses the domain portion as `evil.com` (everything after the last `@`, after the null byte). The email delivery subsystem truncates the string at `%00` and sends to `test@yopmail.com`.

# Proof of Concept
<img width="1919" height="1040" alt="a4" src="https://github.com/user-attachments/assets/978f0837-54b0-4243-bc9d-51f9bd99ad3a" />
<img width="1920" height="1039" alt="a2" src="https://github.com/user-attachments/assets/6d85b5bb-6e79-4806-b461-cf18d939bd07" />
<img width="1920" height="1038" alt="a3" src="https://github.com/user-attachments/assets/4fbc0374-0f6d-4e02-b7d2-c1b2621020a8" />

# Impact
- Security control bypassed
# Recommendation for fix
- **Sanitise and normalise email addresses server-side** before any validation. Strip or reject null bytes (`%00`, `\x00`) and other non-printable characters from the email field before parsing.
- Blocklist check must happen on the **final, sanitised, delivery-ready** email address, not on the raw input string.
