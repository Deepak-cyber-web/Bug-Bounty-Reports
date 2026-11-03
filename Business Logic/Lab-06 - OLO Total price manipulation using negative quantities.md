# Title
Negative Item Price Accepted via Tampered `total` Field in JSON Place-Order Request

# Vulnerability Type
Business Logic Flaw — Insufficient Server-Side Validation of Item Price/Total in JSON Checkout Payload

# Summary
Hacker Café's place-order endpoint accepts a JSON payload containing the full item list with individual `price`, `quantity`, and `total` fields per item. The server does not validate that item totals are positive values. By intercepting the POST request to `/index.php?action=place-order` and changing the `total` field of the Chocolate Brownie from `700` to `-700` (representing -$7.00), the server accepted the tampered payload and the order receipt reflected the Chocolate Brownie as a **-$7.00 line item**. This demonstrates that the server processes client-supplied item totals in the JSON body without sanitising negative values, opening the door to financial manipulation.

# Vulnerable Endpoint
`POST https://upserve.pwnedbox.site/index.php?action=place-order
Content-Type: application/json`

# Steps to Reproduce
1. Log in to Hacker Café as user `gekko`.
2. Add two items to cart:
    - BreadPudding — $9.00
    - Chocolate Brownie — $7.00
    - Checkout total: $19.72 (subtotal $16.00 + tax $1.32 + tip $2.40)
3. Proceed to Checkout, select Pickup, fill in address, set tip to 15%.
4. Enable Burp Suite Intercept (Proxy > Intercept ON).
5. Click **Place Order** — intercept the POST request to `/index.php?action=place-order`.
6. In the JSON body, locate the Chocolate Brownie item object:
    
    ```
    "price": 700, "quantity": 1, "total": 700
    ```
    
7. Change `"total": 700` to `"total": -700` and forward the request.
8. Order receipt (Order hacker-cafe-68312) confirms:
    - 1x BreadPudding @ $9.00 = **$9.00**
    - 1x Chocolate Brownie @ $7.00 = **-$7.00** (negative accepted)
    - Total charged: $19.72

# Payload Used
**Original JSON item object (Chocolate Brownie):**
```json
{
  "item_id": "226b1af0-e309-480a-82fb-d749019ab3ca",
  "name": "Chocolate Brownie",
  "price": 700,
  "quantity": 1,
  "total": 700,
  "modifiers": [],
  "sides": []
}
```

**Modified JSON (exploit):**
```json
{
  "item_id": "226b1af0-e309-480a-82fb-d749019ab3ca",
  "name": "Chocolate Brownie",
  "price": 700,
  "quantity": 1,
  "total": -700,
  "modifiers": [],
  "sides": []
}
```

Change: `"total"` field changed from `700` to `-700` inside the JSON items array.

# Proof of Concept
<img width="1920" height="1036" alt="b1" src="https://github.com/user-attachments/assets/630c0bb9-c9cb-4a5a-9c18-3446804ba0ec" />
<img width="1920" height="1037" alt="b2" src="https://github.com/user-attachments/assets/dd3dd355-ceaf-4f57-a077-b46d5dc9793d" />
<img width="1920" height="1034" alt="b3" src="https://github.com/user-attachments/assets/22bd4cd3-e7b7-4c76-be3a-cd5553065792" />
<img width="1920" height="1041" alt="b4" src="https://github.com/user-attachments/assets/c702660f-80a2-4299-af03-1405ce7f98b4" />


# Impact
- Direct Financial Loss

# Recommendation for fix
- **Never trust item prices or totals in the request payload.** At place-order time, re-fetch all item prices server-side from the menu/product database using `item_id` alone.
