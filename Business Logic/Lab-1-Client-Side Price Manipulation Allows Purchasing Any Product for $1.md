# Title
Client-side price manipulation via Tampered POST Parameter

# Vulnerability Type
Business Logic Flaw — Excessive Trust in Client-Supplied Price Data

# Summary
The STITCHLY e-commerce application passes the product price in the `unit_price_cents` parameter of the POST request when adding items to the cart. The server accepts and uses this value without verifying it against the actual price stored in the database. By intercepting the request in Burp Suite and modifying `unit_price_cents` from `5800` (representing $58.00) to `5` (representing $0.05), the attacker successfully purchased a $58.00 shirt for $0.05.

# Vulnerable Endpoint
`POST https://stitchly.pwnedbox.site/index.php?action=add-to-cart`

# Steps to Reproduce
1. Log in to STITCHLY as user `gekko`.
2. Browse to the Classic Oxford Shirt product page ($58.00, Size XXL).
3. Enable Burp Suite Intercept (Proxy > Intercept ON).
4. Click **Add to Cart** — the POST request to `/index.php?action=add-to-cart` is captured.
5. In the intercepted request body, locate the parameter `unit_price_cents=5800`.
6. Change the value to `unit_price_cents=5` and click Forward.
7. Disable intercept. Navigate to Cart — shirt now shows at **$0.05 each**.
8. Proceed to Checkout, fill in shipping/payment details, and click Place Order.
9. Order confirmation (Order #6) shows: **1x Classic Oxford Shirt (XXL) @ $0.05 = $0.05**. Total charged: $8.05 (item $0.05 + shipping $8.00).

# Payload Used

**Original POST body:**
```
product_id=1&unit_price_cents=5800&size=XXL&qty=1
```

**Modified POST body (exploit):**
```
product_id=1&unit_price_cents=5&size=XXL&qty=1
```

Change: `unit_price_cents` reduced from `5800` ($58.00) to `5` ($0.05).

# Proof of Concept
<img width="1920" height="1036" alt="b1" src="https://github.com/user-attachments/assets/bbdc8a55-5494-43e3-ba48-55c88539536a" />
<img width="1920" height="1035" alt="b2" src="https://github.com/user-attachments/assets/7da39a38-a5f9-4eb2-9408-0929490848eb" />
<img width="1920" height="1039" alt="b3" src="https://github.com/user-attachments/assets/ec423b31-1669-4f50-b7b6-54113cfcd78d" />
<img width="1920" height="1043" alt="b4" src="https://github.com/user-attachments/assets/32327634-39ae-46db-b5a4-13b3eaef2cc7" />
<img width="1920" height="1046" alt="b5" src="https://github.com/user-attachments/assets/77bb4d25-c85e-4959-aaf1-14ccc4a06467" />

# Impact
- **Direct Financial Loss** - Attacker purchases a $58.00 product for $0.05 — 99.9% discount
- **Scalability** - Applies to every product in the store; any item can be set to any price including $0.00
# Recommendation for fix
- **Never accept price from the client.** Remove `unit_price_cents` from the POST body entirely. The server must look up the price from the database using `product_id` alone.
- Apply server-side validation: at the point of cart addition AND at checkout, compare the cart price against the database price and reject mismatches.
- Treat all client-supplied values for financial fields (price, discount, quantity cap) as untrusted input.
