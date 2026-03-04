---
name: order
description: >
  Place orders on Bob's Online Store. Use when the user wants to add items to a cart,
  apply coupons, choose shipping, and complete checkout via API. Handles the
  full purchase workflow from cart to confirmation. Requires a Bearer token
  (user must be logged in). Do NOT use for product search or browsing.
version: 1.0.0
auth: bearer
rate_limit:
  agent: 5/minute
  api: 20/minute
base_url: https://api.bobs-store.com/v1
---

# Place Order

## Authentication

Include the user's Bearer token in every request:

```
Authorization: Bearer USER_ACCESS_TOKEN
```

Users obtain tokens via OAuth 2.0 at https://bobs-store.com/oauth/authorize.

## Workflow

Complete the following steps in order.

### Step 1: Add Items to Cart

POST /cart/items

```json
{
  "product_id": "prod_8x7k",
  "quantity": 1
}
```

Response (201 Created):

```json
{
  "cart_id": "cart_r4t9",
  "items": [
    {
      "product_id": "prod_8x7k",
      "name": "SoundWave Pro Wireless Headphones",
      "quantity": 1,
      "unit_price": 79.99,
      "subtotal": 79.99
    }
  ],
  "total": 79.99,
  "currency": "USD"
}
```

Repeat this step to add multiple products. Each call adds one item.

### Step 2: Apply Coupon (optional)

POST /cart/coupon

```json
{
  "code": "SAVE20"
}
```

Response (200 OK):

```json
{
  "cart_id": "cart_r4t9",
  "coupon": "SAVE20",
  "discount": 16.00,
  "total": 63.99,
  "currency": "USD"
}
```

Only one coupon per cart. To change, send a new code — it replaces the previous one.

### Step 3: Create Order

POST /orders

```json
{
  "cart_id": "cart_r4t9",
  "shipping_method": "standard",
  "shipping_address": {
    "name": "Jane Doe",
    "line1": "123 Main St",
    "city": "San Francisco",
    "state": "CA",
    "zip": "94105",
    "country": "US"
  },
  "payment_token": "tok_visa_4242"
}
```

| Field            | Type   | Required | Description                                |
|------------------|--------|----------|--------------------------------------------|
| cart_id          | string | yes      | Cart ID from Step 1                        |
| shipping_method  | string | yes      | `standard` (5-7 days), `express` (2-3 days), `overnight` |
| shipping_address | object | yes      | Delivery address                           |
| payment_token    | string | yes      | Tokenized payment from client-side SDK     |

Response (201 Created):

```json
{
  "order_id": "ord_7f2k",
  "status": "confirmed",
  "total": 63.99,
  "shipping_method": "standard",
  "estimated_delivery": "2026-03-11",
  "created_at": "2026-03-04T10:30:00Z"
}
```

### Step 4: Check Order Status

GET /orders/{order_id}

Response (200 OK):

```json
{
  "order_id": "ord_7f2k",
  "status": "shipped",
  "tracking_number": "1Z999AA10123456784",
  "carrier": "UPS",
  "estimated_delivery": "2026-03-11",
  "items": [
    {
      "product_id": "prod_8x7k",
      "name": "SoundWave Pro Wireless Headphones",
      "quantity": 1,
      "unit_price": 79.99
    }
  ],
  "total": 63.99
}
```

Possible status values: `confirmed` → `processing` → `shipped` → `delivered`

## Error Handling

| Code | Meaning                | Action                                  |
|------|------------------------|-----------------------------------------|
| 400  | Invalid request        | Check required fields and types         |
| 401  | Auth failed            | User needs to re-authenticate           |
| 404  | Cart/order not found   | Verify cart_id or order_id              |
| 409  | Cart already ordered   | Cannot reuse a cart; create a new one   |
| 422  | Payment declined       | Ask user to try a different payment method |
| 429  | Rate limited           | Wait, retry after `Retry-After` header  |
| 500  | Server error           | Retry with exponential backoff          |

## Notes

- **Idempotency:** POST /orders supports an `Idempotency-Key` header. Include one to safely retry on network failures.
- **Payment tokens:** Never store raw card numbers. Use the Bob's Online Store client-side SDK to tokenize payment details.
- **Cancellation window:** Orders can be cancelled within 30 minutes of creation via DELETE /orders/{order_id}.
- **Cart expiry:** Carts expire after 24 hours of inactivity.
