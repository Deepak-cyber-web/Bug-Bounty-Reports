# Title
Unlimited Coupon Stacking via Repeated POST Requests to Apply-Coupon Endpoint

# Vulnerability Type
Business Logic Flaw — Flawed Enforcement of Single-Use Coupon Rule (No Server-Side Rate/Repeat Check)

# Summary
The PCHub e-commerce application allows users to apply a discount coupon via a POST request to `/subdomains/controllers/cart/apply-coupon`. The UI states _"Only one coupon can be used per order"_ — but the server does not enforce this rule when the same request is replayed. By sending the apply-coupon request multiple times using Burp Repeater, the discount stacks on each replay, growing far beyond the intended one-time discount. The coupon WELCOME20 (intended: ₹3,000 off) was stacked to a ₹11,853 discount on a ₹14,999 item by replaying the request in Repeater, reducing the final payable amount from ₹21,899 to ₹13,046.

# Vulnerable Endpoint
`POST https://controllers.pwnedbox.site/subdomains/controllers/cart/apply-coupon`

# Steps to Reproduce
1. Log in to PCHub as user `gekko`.
2. Add Samsung 990 Pro 2TB NVMe (₹14,999) to cart.
3. Apply coupon code `WELCOME20` via the Coupons & Offers section.
4. Confirm: "Coupon WELCOME20 applied successfully" — discount shows ₹3,000, total ₹21,899.
5. In Burp Suite, go to **HTTP History**, find the POST request to `/subdomains/controllers/cart/apply-coupon`.
6. Right-click → **Send to Repeater**.
7. In Repeater, click **Send** repeatedly without changing any parameter.
8. After each send, observe the cart — coupon discount keeps increasing:
    - After 1st replay: discount = **₹5,400**, total = ₹19,499
    - After more replays: discount = **₹11,853**, total = ₹13,046
9. Once satisfied, proceed to checkout → payment page confirms **Amount Due: ₹13,046** (COD).
10. Confirm COD order — Order #1006 placed.

# Payload Used
**Request replayed in Burp Repeater (no modification needed):**

```
POST /subdomains/controllers/cart/apply-coupon HTTP/2
Host: controllers.pwnedbox.site
Cookie: PCHUBBID=<session>
Content-Type: application/x-www-form-urlencoded

coupon_code=WELCOME20&_csrf=<token>
```

No parameter change required — the same request is replayed multiple times. Each replay increases the applied discount.

# Proof of Concept
<img width="1920" height="1042" alt="a1" src="https://github.com/user-attachments/assets/a2a7f19d-9f29-4606-b709-70a26dd07f45" />
<img width="1920" height="1033" alt="a4" src="https://github.com/user-attachments/assets/3a2df2b4-adba-4814-999d-c21010bfdbf6" />
<img width="1920" height="1032" alt="a5" src="https://github.com/user-attachments/assets/5638c01c-e5c6-449b-98bc-8c291b04116c" />


# Impact
- Direct Financial Loss

# Recommendation for fix
- **Track coupon application state server-side.** Once a coupon is applied to a cart/order, mark it as applied and reject any further apply requests for the same coupon on the same cart session.

