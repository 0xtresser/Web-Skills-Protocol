---
name: search
description: >
  Search and browse products in Bob's Online Store catalog.
  Use when the user wants to find products by keyword, category, price range,
  or brand. Returns structured product data including prices, ratings,
  and stock availability. Supports pagination and sorting.
  Do NOT use for order management or account operations.
version: 1.0.0
auth: none
rate_limit:
  agent: 30/minute
  api: 100/minute
base_url: https://api.bobs-store.com/v1
---

# Product Search

## Note

This endpoint is publicly accessible — no authentication required.

## Endpoint

GET /products

## Parameters

| Parameter  | Type   | Required | Description                              |
|------------|--------|----------|------------------------------------------|
| q          | string | yes      | Search query                             |
| category   | string | no       | Filter by category (see Categories below)|
| min_price  | number | no       | Minimum price in USD                     |
| max_price  | number | no       | Maximum price in USD                     |
| brand      | string | no       | Filter by brand name (exact match)       |
| in_stock   | bool   | no       | Only show in-stock items (default: false)|
| sort       | string | no       | Sort by: `relevance`, `price_asc`, `price_desc`, `rating`, `newest` |
| page       | number | no       | Page number (default: 1)                 |
| per_page   | number | no       | Results per page, max 50 (default: 20)   |

## Categories

Valid category values:

- `audio` — Headphones, speakers, microphones
- `computers` — Laptops, desktops, monitors
- `phones` — Smartphones, cases, chargers
- `gaming` — Consoles, controllers, accessories
- `cameras` — Cameras, lenses, tripods
- `smart-home` — Smart speakers, lights, thermostats
- `wearables` — Smartwatches, fitness trackers

## Example

Request:

```
GET /products?q=wireless+headphones&sort=rating&max_price=100&in_stock=true
```

Response (200 OK):

```json
{
  "products": [
    {
      "id": "prod_8x7k",
      "name": "SoundWave Pro Wireless Headphones",
      "brand": "SoundWave",
      "price": 79.99,
      "category": "audio",
      "rating": 4.7,
      "review_count": 1243,
      "in_stock": true,
      "image_url": "https://bobs-store.com/images/prod_8x7k.jpg",
      "url": "https://bobs-store.com/products/prod_8x7k"
    },
    {
      "id": "prod_m3nq",
      "name": "BassX Foldable Bluetooth Headphones",
      "brand": "BassX",
      "price": 49.99,
      "category": "audio",
      "rating": 4.3,
      "review_count": 876,
      "in_stock": true,
      "image_url": "https://bobs-store.com/images/prod_m3nq.jpg",
      "url": "https://bobs-store.com/products/prod_m3nq"
    }
  ],
  "total": 42,
  "page": 1,
  "per_page": 20
}
```

## Error Handling

| Code | Meaning             | Action                              |
|------|---------------------|-------------------------------------|
| 400  | Invalid parameters  | Check parameter types and values    |
| 429  | Rate limited        | Wait, retry after `Retry-After` header |
| 500  | Server error        | Retry with exponential backoff      |

## Notes

- Search is case-insensitive and supports partial matching.
- Empty `q` with a `category` filter returns top products in that category.
- Results are deduplicated — the same product won't appear twice.
- Pagination: use `page` and `per_page`. The response `total` field shows total matches.
