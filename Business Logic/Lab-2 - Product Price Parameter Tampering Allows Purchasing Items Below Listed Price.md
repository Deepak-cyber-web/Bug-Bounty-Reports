# Title
Cart Total Manipulation via Negative Quantity in Cart Update Request

# Vulnerability Type
Business Logic Flaw — Insufficient Input Validation on Quantity Parameter (Negative Value Accepted)

# Summary
The ControlZone e-commerce application allows users to update item quantities in the cart via a POST request to `/cart/update`. The server does not validate that the `quantity` parameter is a positive integer. By intercepting the update request and setting `quantity=-1` for an item, the server treats the item's price as a negative value and subtracts it from the cart total. This allowed purchase of a ₹2,299 gaming controller for only ₹150 by using a second item as an effective ₹2,149 discount.

# Vulnerable Endpoint
`POST https://controlzone.pwnedbox.site/cart/update`

# Steps to Reproduce
1. Log in to ControlZone as user `gekko`.
2. Add two items to the cart:
    - NiTHO V80 Bluetooth Gamepad — ₹2,299 (qty: 1)
    - Zebronics Max Link Pro Wireless Controller — ₹2,149 (qty: 2)
    - Cart total: ₹4,448
3. Enable Burp Suite Intercept (Proxy > Intercept ON).
4. Click **Update** on the Zebronics item — intercept the POST request to `/cart/update`.
5. In the request body, locate `product_id=3&quantity=2`.
6. Change `quantity` to `-1` and forward the request.
7. Cart now shows: Zebronics qty = -1, subtotal = **-₹2,149** (displayed in red).
8. New cart total: ₹2,299 + (-₹2,149) = **₹150**.
9. Proceed to checkout → Payment page shows Order #1006 Amount Due: **₹150**.
10. Complete payment — order confirmed. Paid ₹150 for a ₹2,299 item.

# Payload Used
**Original POST body:**
```
_csrf=<token>&product_id=3&quantity=2
```

**Modified POST body (exploit):**
```
_csrf=<token>&product_id=3&quantity=-1
```

Change: `quantity` set to `-1` — server accepts it, applies negative price arithmetic to cart total.

# Proof of Concept
<img width="1920" height="1039" alt="a1" src="https://github.com/user-attachments/assets/a2631fa6-f7b7-4326-92e6-6db6726341ac" />
<img width="1920" height="1040" alt="a2" src="https://github.com/user-attachments/assets/c6caa987-33f5-4b93-976d-b267b95de04b" />
<img width="1920" height="1035" alt="a3" src="https://github.com/user-attachments/assets/24baa8c4-c056-4eb8-a4af-d8e526882c51" />
<img width="1920" height="1028" alt="a4" src="https://github.com/user-attachments/assets/5fba3654-13b5-4f1b-98b9-335ffe5889be" />
<img width="1920" height="1033" alt="a5" src="https://github.com/user-attachments/assets/51d854a9-71ee-40ac-abaa-d451ebfedd1c" />


# Impact
- **Direct Financial Loss**
# Recommendation for fix
- **Server-side validation:** Reject any `quantity` value that is zero or negative — return HTTP 400 with an error message.
- Enforce minimum quantity of 1 at the business logic layer, not just the frontend UI.
