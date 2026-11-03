# Title
Product Price Manipulation via Tampered `price` Parameter in Add-to-Cart POST Request

# Vulnerability Type
Business Logic Flaw — Excessive Trust in Client-Supplied Price Parameter on Cart Add Endpoint

# Summary
RetailZone's add-to-cart flow sends the product price as a client-controlled `price` parameter in the POST body. The server accepts this value without validating it against the actual price stored in the database. By intercepting the POST request to `/subdomains/retailzone/cart/add` and changing `price=699` to `price=9`, a ₹699 product (Sunscreen SPF 50+ PA++++) was added to the cart at ₹9 — a 99% discount — and the order was successfully placed and marked as delivered for a total payment of ₹108 (₹9 item + ₹99 delivery).

# Vulnerable Endpoint
`POST https://retailzone.pwnedbox.site/subdomains/retailzone/cart/add`

# Steps to Reproduce
1. Log in to RetailZone and browse to the Sunscreen SPF 50+ PA++++ product page (`/product/19`).
    - Listed price: ₹699 (22% off from ₹899)
2. Enable Burp Suite Intercept (Proxy > Intercept ON).
3. Click **Add to Bag** — intercept the POST request to `/subdomains/retailzone/cart/add`.
4. In the request body, locate the parameter `price=699`.
5. Change it to `price=9` and forward the request.
6. Cart (My Bag) now shows: Sunscreen SPF 50+ PA++++ at **₹9 — 99% OFF**.
    - Subtotal: ₹9 | Delivery: ₹99 | Total: ₹108
7. Proceed to checkout and place the order.
8. Order #1005 confirmed — Total Paid: **₹108**. Order instantly marked as **Delivered**.

# Payload Used
**Original POST body:**
```
_csrf=<token>&product_id=19&price=699&size=One+Size
```

**Modified POST body (exploit):**
```
_csrf=<token>&product_id=19&price=9&size=One+Size
```

Change: `price` reduced from `699` to `9`. Any arbitrary value is accepted by the server.

# Proof of Concept
<img width="1920" height="1042" alt="a1" src="https://github.com/user-attachments/assets/bc54582f-9c6c-409b-b945-64d6f10c6d46" />
<img width="1920" height="1041" alt="a2" src="https://github.com/user-attachments/assets/374758ee-cdf0-4351-a805-c78b1ebf519d" />
<img width="1920" height="1044" alt="a3" src="https://github.com/user-attachments/assets/ee938a4b-7a25-4cbe-821c-eb26c7ba01e2" />
<img width="1920" height="1034" alt="a4" src="https://github.com/user-attachments/assets/62feec61-aeef-4fab-9d3a-2b20330da23c" />


# Impact
- Direct Financial Loss

# Recommendation for fix
 - **Never accept price from the client.** Remove `unit_price_cents` from the POST body entirely. The server must look up the price from the database using `product_id` alone.
- Apply server-side validation: at the point of cart addition AND at checkout, compare the cart price against the database price and reject mismatches.
