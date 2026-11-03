# Title
Marketplace Article Price Reduction via Negative Quantity Value in JSON Bulk Purchase Request

# Vulnerability Type
Business Logic Flaw — Negative Quantity Accepted in JSON Payload; Server Recalculates Total from Tampered Item Quantities

# Summary
ContentForge's bulk purchase endpoint accepts a JSON payload where article quantities are passed as integer values keyed by article type (e.g. `"article_500": 1`). The server does not validate that these quantity values are positive integers. By intercepting the POST request to `/index.php?action=purchases-bulk` and changing `"article_500": 1` to `"article_500": -1`, the server treated the 500 Words article as a -1 quantity purchase — subtracting its $40.00 price from the total. The order was placed for **$30.00 instead of $110.00**, and wallet balance confirmed only $30 was deducted ($300 → $270). Unlike similar labs, the server here **recalculated the total from the tampered payload**, making this a confirmed full-impact exploit.

# Vulnerable Endpoint
`POST https://marketplace.pwnedbox.site/index.php?action=purchases-bulk
Content-Type: application/json`

# Steps to Reproduce
1. Log in to ContentForge as user `Lovable` — wallet balance: **$300.00**.
2. Add two articles to the order via Order Builder:
    - 1000 Words — $70.00 each
    - 500 Words — $40.00 each
    - Order total: **$110.00**
3. Navigate to Review Order page (`/index.php?action=review-order`).
4. Enable Burp Suite Intercept (Proxy > Intercept ON).
5. Click **Confirm & Pay** — intercept the POST request to `/index.php?action=purchases-bulk`.
6. In the JSON body, locate the items object:
    
    ```json
    {"items": {"article_1000": 1, "article_500": 1}}
    ```
    
7. Change `"article_500": 1` to `"article_500": -1` and forward the request.
8. Order receipt (Order #9) confirms:
    - 1x 1000 Words @ $70.00 = **$70.00**
    - -1x 500 Words @ $40.00 = **-$40.00**
    - **Total charged: $30.00**
    - Wallet balance after: **$270.00** (only $30 deducted from $300)

# Payload Used
**Original JSON body:**
```json
{"items": {"article_1000": 1, "article_500": 1}}
```

**Modified JSON body (exploit):**
```json
{"items": {"article_1000": 1, "article_500": -1}}
```

Change: quantity value for `article_500` changed from `1` to `-1`. The integer value of each key in the `items` object represents quantity — not explicitly labelled, but functionally equivalent.

# Proof of Concept
<img width="1920" height="1030" alt="b1" src="https://github.com/user-attachments/assets/c5401139-1279-4d6b-b433-5c599abee292" />
<img width="1920" height="1035" alt="b2" src="https://github.com/user-attachments/assets/4671654b-b6b5-4784-92b2-38b974211a06" />
<img width="1920" height="1043" alt="b3" src="https://github.com/user-attachments/assets/4d663b8d-9a6a-49fd-97e6-a2146818cc4d" />


# Impact
- Financial Loss

# Recommendation for fix
- **Server-side validation:** Reject any `quantity` value that is zero or negative — return HTTP 400 with an error message.
- Enforce minimum quantity of 1 at the business logic layer, not just the frontend UI.
