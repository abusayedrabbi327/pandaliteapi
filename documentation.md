# Panda-Lite — Full API Documentation

## UPDATE 1: Add `GET /_internal/db-state` endpoint.

## This project has some intentional(!) issues or incomplete features.

## What It Is

Panda-Lite is a lightweight food-delivery REST API built with Node.js and Express, designed as a backend for the STQA Gateway (see the gateway's own README for provisioning and routing). It models the core workflow of a delivery platform: customers order from restaurants, restaurants prepare orders, and riders deliver them. Orders move through a defined lifecycle:

```
placed → accepted → preparing → ready_for_pickup → picked_up → delivered
```

Orders can also be **cancelled**, but only from `placed` or `accepted` — see [Order Lifecycle](#order-lifecycle) for the rules.

All data is stored in **Postgres**, in a team-specific database selected via the signed gateway context (`X-STQA-Context`). Panda-Lite does not manage its own database credentials or team routing — see the gateway README, section 6, for how the backend receives `databaseName` and connects.

---

## Tech Stack

| Piece | Detail |
|---|---|
| Runtime | Node.js |
| Framework | Express 5 |
| Database | PostgreSQL (per-team database, provided by the gateway) |
| App Auth | JWT (jsonwebtoken), 24h expiry |
| Gateway Auth | Signed `X-STQA-Context` header, verified with `CONTEXT_SIGNING_SECRET` |
| Passwords | bcryptjs (10 salt rounds) |
| IDs | `crypto.randomUUID()` — UUIDs everywhere |
| Port | `4000` (override with `PORT` env var) |
| App JWT Secret | `dev_app_jwt_secret_change_me` (override with `APP_JWT_SECRET` env var) |

---

## How to Run

This service is not meant to be run standalone against the public internet — it is designed to sit behind the STQA Gateway, which handles student identity, team isolation, and routing. See the gateway README for how to start the full stack (`docker compose up`) and how requests reach this service.

### 1. Install dependencies

```bash
npm install
```

### 2. Start the server

```bash
# Production
npm start

# Development (auto-restarts on file changes — requires nodemon)
npm run dev
```

The server will print:
```
Server running on port 7000
```

### 3. Health check

```
GET /_internal/health
```
```json
{ "status": "healthy", "version": "panda-lite-v1" }
```

### 4. Reset database

Wipes **everything** (users, restaurants, menu items, orders, order items, ratings, timeline events) for the team database selected by the `X-STQA-Context` header. Intended for test setup/teardown, not for use against real data.

```
POST /_internal/reset-database
```
```json
{ "status": "reset" }
```

---

### 5. Inspect database state

Returns a full snapshot of the team database selected by the `X-STQA-Context` header — every user, restaurant, menu item, order (with its `items` and `timeline`), and rating. Intended for test verification/debugging, not part of the app-facing API (no `Authorization` bearer token required).

```
GET /_internal/db-state
```
```json
{
  "users": [ /* User objects, see Data Models */ ],
  "restaurants": [ /* Restaurant objects */ ],
  "menuItems": [ /* MenuItem objects */ ],
  "orders": [ /* Order objects, each including nested items and timeline */ ],
  "ratings": [ /* Rating objects */ ]
}
```

---
### 6. Assignment-Specific Notes

#### X-STQA-Key
- You should have received a secret key via email.
- Include this key in the header of all your requests (`X-STQA-Key`).
- Keep this key private; do not share it or send requests using another student's key.

#### `_lab` Field
- Each response will include a `_lab` field containing a unique request ID. Please include these IDs in your report for reference.

```json
{
    "other fields": "...",
    "_lab": {
        "requestId": "uuid"
    }
}
```


## Authentication

Panda-Lite has **two independent auth layers**:

1. **Gateway layer** — every request arrives with a signed `X-STQA-Context` header, verified against `CONTEXT_SIGNING_SECRET`. This identifies the student/team and selects the Postgres database. This happens before any route logic runs and is not part of the API surface described below.
2. **App layer** — Panda-Lite's own login system, separate from the gateway. Every endpoint except `POST /auth/register` and `POST /auth/login` requires a Bearer token issued by `POST /auth/login`.

```
Authorization: Bearer <token>
```

Tokens are valid for **24 hours**. A missing, invalid, or expired token returns `401`.

---

## Data Models

### User
```json
{
  "id": "uuid",
  "name": "Alice Rider",
  "email": "alice@example.com",
  "role": "customer | restaurant | rider",
  "createdAt": "2026-01-01T00:00:00.000Z"
}
```
> `passwordHash` exists on the internal row but is **never returned** by any endpoint.

### Restaurant
```json
{
  "id": "uuid",
  "ownerId": "uuid",
  "name": "Dhaka Biryani House",
  "address": "House 12, Road 5, Dhanmondi",
  "isOpen": true,
  "createdAt": "2026-01-01T00:00:00.000Z"
}
```
> `ownerId` must belong to a user with role `restaurant`.

### MenuItem
```json
{
  "id": "uuid",
  "restaurantId": "uuid",
  "name": "Chicken Biryani",
  "price": 250,
  "stockQuantity": 20,
  "isAvailable": true,
  "createdAt": "2026-01-01T00:00:00.000Z"
}
```
> `isAvailable` is `false` automatically when `stockQuantity` reaches `0`.

### Order
```json
{
  "id": "uuid",
  "customerId": "uuid",
  "restaurantId": "uuid",
  "riderId": "uuid | null",
  "status": "placed",
  "subtotal": 500,
  "deliveryFee": 50,
  "platformFee": 50,
  "total": 600,
  "cancelReason": "string | null",
  "createdAt": "2026-01-01T00:00:00.000Z",
  "updatedAt": "2026-01-01T00:00:00.000Z"
}
```
> `riderId` is `null` until a rider claims the order.
> `total = subtotal + deliveryFee + platformFee`. `deliveryFee` is a flat amount; `platformFee` is a percentage of `subtotal`. See [Environment Variables](#environment-variables) for the configured rate.

### OrderItem
```json
{
  "id": "uuid",
  "orderId": "uuid",
  "menuItemId": "uuid",
  "nameSnapshot": "Chicken Biryani",
  "priceSnapshot": 250,
  "quantity": 2
}
```
> `nameSnapshot`/`priceSnapshot` freeze the item's name/price at order time, so later menu edits don't change historical orders.

### Rating
```json
{
  "id": "uuid",
  "orderId": "uuid",
  "customerId": "uuid",
  "target": "restaurant | rider",
  "targetId": "uuid",
  "score": 5,
  "comment": "Food was great and arrived hot!",
  "createdAt": "2026-01-01T00:00:00.000Z"
}
```
> Each order can produce **up to two** ratings from the customer: one for the restaurant, one for the rider. `score` is an integer 1–5.

### OrderTimelineEvent
```json
{
  "id": "uuid",
  "orderId": "uuid",
  "status": "accepted",
  "actorId": "uuid",
  "actorRole": "restaurant",
  "occurredAt": "2026-01-01T00:05:00.000Z"
}
```
> One event is recorded every time an order's status changes — including `placed` (recorded at creation, `actorRole: "customer"`) and `cancelled`. Claiming an order also records a `claimed` timeline event even though it does not change `status` itself (see `PATCH /orders/:id/claim` below) — `status` remains `ready_for_pickup` until the rider marks it `picked_up`.

---

## Order Lifecycle

```
placed ──[accept]──► accepted ──[prepare]──► preparing ──[mark ready]──► ready_for_pickup
  │                     │
  │                     │
  └──[cancel, free]─────┘──[cancel, penalty]──► cancelled
                                                      ▲
ready_for_pickup ──[claim]──► picked_up ──[deliver]──► delivered
```

| Action | Endpoint | Who can do it | Allowed from |
|---|---|---|---|
| Accept | `PATCH /orders/:id/status` (`accepted`) | Restaurant owner only | `placed` |
| Start preparing | `PATCH /orders/:id/status` (`preparing`) | Restaurant owner only | `accepted` |
| Mark ready | `PATCH /orders/:id/status` (`ready_for_pickup`) | Restaurant owner only | `preparing` |
| Claim | `PATCH /orders/:id/claim` | Any rider (first to claim) | `ready_for_pickup` |
| Mark picked up | `PATCH /orders/:id/status` (`picked_up`) | Assigned rider only | `ready_for_pickup` |
| Mark delivered | `PATCH /orders/:id/status` (`delivered`) | Assigned rider only | `picked_up` |
| Cancel (free) | `PATCH /orders/:id/cancel` | Customer (order owner) only | `placed` |
| Cancel (penalty) | `PATCH /orders/:id/cancel` | Customer (order owner) only | `accepted` |

> Cancelling from `placed` is free. Cancelling from `accepted` is still allowed but applies a cancellation penalty (see `POST /orders/:id/cancel` below for the exact response shape). **Orders cannot be cancelled once `preparing` or later.**

---

## Endpoints

### Shared List Query Parameters

The list endpoints below support a shared query contract for sorting and exact field-value filtering.

| Param | Type | Description |
|---|---|---|
| `sort_by` | `asc \| desc` | Sort direction for the endpoint's default sort field. |
| `filter_field` | string | Field name to filter on (must be in the endpoint's allowed field list). |
| `filter_value` | string | Exact value to match for `filter_field` (case-insensitive for text fields). |

Validation rules:

- `sort_by` must be `asc` or `desc`.
- `filter_field` and `filter_value` must be provided together.
- Invalid `filter_field` returns `400` with the allowed field list.

Type matching notes:

- Boolean filters accept `true` or `false`.
- Number filters require an exact numeric match.
- Text filters are case-insensitive exact matches.

---

### Auth

#### `POST /auth/register`

Register a new user.

**Body:**
```json
{
  "name": "Alice Rider",
  "email": "alice@example.com",
  "password": "pass123",
  "role": "customer"
}
```
> `role` must be one of `customer`, `restaurant`, `rider`.

**Success — 201:**
```json
{
  "id": "a1b2c3d4-...",
  "name": "Alice Rider",
  "email": "alice@example.com",
  "role": "customer"
}
```

**Failures:**

| Condition | Status | Response |
|---|---|---|
| Missing any field | 400 | `{ "error": "name, email, password and role are required" }` |
| Invalid role | 400 | `{ "error": "role must be one of customer, restaurant, rider" }` |
| Email already used | 409 | `{ "error": "Email already in use" }` |
| Password must be at least 6 characters long | 400 | `{ "error": "Password must be at least 6 characters long" }` |

---

#### `POST /auth/login`

Log in and receive a JWT.

**Body:**
```json
{ "email": "alice@example.com", "password": "pass123" }
```

**Success — 200:**
```json
{ "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." }
```

**Failures:**

| Condition | Status | Response |
|---|---|---|
| Missing email or password | 400 | `{ "error": "email and password are required" }` |
| Email not found | 401 | `{ "error": "Invalid credentials" }` |
| Wrong password | 401 | `{ "error": "Invalid credentials" }` |

---

### Restaurants

#### `GET /restaurants`

List all open restaurants.

**Query params:**

| Param | Description |
|---|---|
| `name` | Filter by name (case-insensitive, partial match) |
| `sort_by` | `asc` or `desc` (default sort field: `name`) |
| `filter_field` | One of: `id`, `ownerId`, `name`, `address`, `isOpen`, `createdAt` |
| `filter_value` | Exact value for `filter_field` |

**Examples:**

```http
GET /restaurants?sort_by=desc
GET /restaurants?filter_field=isOpen&filter_value=true
GET /restaurants?name=biryani&sort_by=asc&filter_field=address&filter_value=house%2012
```

**Success — 200:** Returns an array of restaurant objects. Only restaurants with `isOpen: true` are included.

**Failures:**

| Condition | Status | Response |
|---|---|---|
| No token | 401 | `{ "error": "Missing token" }` |
| Invalid list query params | 400 | e.g. `{ "error": "sort_by must be either 'asc' or 'desc'" }` |

---

#### `GET /restaurants/:id`

Get a single restaurant by ID.

**Success — 200:** Returns the restaurant object, regardless of `isOpen` status.

**Failures:**

| Condition | Status | Response |
|---|---|---|
| No token | 401 | `{ "error": "Missing token" }` |
| Restaurant not found | 404 | `{ "error": "Restaurant not found" }` |

---

#### `POST /restaurants`

Create a restaurant. The authenticated user becomes the owner.

**Body:**
```json
{ "name": "Dhaka Biryani House", "address": "House 12, Road 5, Dhanmondi" }
```

**Success — 201:** Returns the new restaurant object (`isOpen: true` by default).

**Failures:**

| Condition | Status | Response |
|---|---|---|
| No token | 401 | `{ "error": "Missing token" }` |
| Requester's role is not `restaurant` | 403 | `{ "error": "Only restaurant accounts can create a restaurant" }` |
| Missing name or address | 400 | `{ "error": "name and address are required" }` |

---

#### `PATCH /restaurants/:id`

Update a restaurant's own details (name, address, or open/closed status).

**Body (any combination):**
```json
{ "name": "New Name", "address": "New Address", "isOpen": false }
```

**Success — 200:** Returns the updated restaurant object.

**Failures:**

| Condition | Status | Response |
|---|---|---|
| No token | 401 | `{ "error": "Missing token" }` |
| Restaurant not found | 404 | `{ "error": "Restaurant not found" }` |
| Requester's `id` does not match the restaurant's `ownerId` | 403 | `{ "error": "Only the restaurant owner can update this restaurant" }` |

---

### Menu Items

#### `GET /restaurants/:id/menu`

List all menu items for a restaurant.

**Query params:**

| Param | Description |
|---|---|
| `sort_by` | `asc` or `desc` (default sort field: `createdAt`) |
| `filter_field` | One of: `id`, `restaurantId`, `name`, `price`, `stockQuantity`, `isAvailable`, `createdAt` |
| `filter_value` | Exact value for `filter_field` |

**Examples:**

```http
GET /restaurants/:id/menu?sort_by=desc
GET /restaurants/:id/menu?filter_field=isAvailable&filter_value=true
GET /restaurants/:id/menu?filter_field=price&filter_value=250&sort_by=asc
```

**Success — 200:** Returns an array of menu item objects, including unavailable ones (`isAvailable: false`).

**Failures:**

| Condition | Status | Response |
|---|---|---|
| No token | 401 | `{ "error": "Missing token" }` |
| Restaurant not found | 404 | `{ "error": "Restaurant not found" }` |
| Invalid list query params | 400 | e.g. `{ "error": "filter_field must be one of: id, restaurantId, name, price, stockQuantity, isAvailable, createdAt" }` |

---

#### `POST /restaurants/:id/menu`

Add a menu item. Only the restaurant's owner can do this.

**Body:**
```json
{ "name": "Chicken Biryani", "price": 250, "stockQuantity": 20 }
```

**Success — 201:** Returns the new menu item object. `isAvailable` is set to `true` if `stockQuantity > 0`, else `false`.

**Failures:**

| Condition | Status | Response |
|---|---|---|
| No token | 401 | `{ "error": "Missing token" }` |
| Restaurant not found | 404 | `{ "error": "Restaurant not found" }` |
| Requester's `id` does not match the restaurant's `ownerId` | 403 | `{ "error": "Only the restaurant owner can manage the menu" }` |
| Missing name or price | 400 | `{ "error": "name and price are required" }` |
| Negative price or stockQuantity | 400 | `{ "error": "price and stockQuantity must be non-negative" }` |

---

#### `PATCH /menu-items/:id`

Update a menu item (price, stock, availability). Only the owning restaurant's owner can do this.

**Body (any combination):**
```json
{ "price": 275, "stockQuantity": 15, "isAvailable": true }
```

**Success — 200:** Returns the updated menu item object.

**Failures:**

| Condition | Status | Response |
|---|---|---|
| No token | 401 | `{ "error": "Missing token" }` |
| Menu item not found | 404 | `{ "error": "Menu item not found" }` |
| Requester's `id` does not match the owning restaurant's `ownerId` | 403 | `{ "error": "Only the restaurant owner can manage the menu" }` |
| Negative price or stockQuantity | 400 | `{ "error": "price and stockQuantity must be non-negative" }` |

---

### Orders

#### `POST /orders`

Place a new order with a single restaurant. Only customers can place orders.

**Body:**
```json
{
  "restaurantId": "r1r2r3-...",
  "items": [
    { "menuItemId": "m1m2m3-...", "quantity": 2 },
    { "menuItemId": "m4m5m6-...", "quantity": 1 }
  ]
}
```

**Success — 201:**
```json
{
  "id": "o1o2o3-...",
  "customerId": "a1b2c3-...",
  "restaurantId": "r1r2r3-...",
  "riderId": null,
  "status": "placed",
  "subtotal": 500,
  "deliveryFee": 50,
  "platformFee": 50,
  "total": 600,
  "items": [
    { "menuItemId": "m1m2m3-...", "nameSnapshot": "Chicken Biryani", "priceSnapshot": 250, "quantity": 2 }
  ],
  "timeline": [
    { "status": "placed", "actorId": "a1b2c3-...", "actorRole": "customer", "occurredAt": "2026-01-01T00:00:00.000Z" }
  ],
  "createdAt": "2026-01-01T00:00:00.000Z"
}
```
> Placing an order decrements `stockQuantity` on each ordered menu item. If `stockQuantity` reaches `0`, `isAvailable` is set to `false`. Placing an order also creates the first `timeline` entry (`status: "placed"`).

**Failures:**

| Condition | Status | Response |
|---|---|---|
| No token | 401 | `{ "error": "Missing token" }` |
| Requester's role is not `customer` | 403 | `{ "error": "Only customer accounts can place orders" }` |
| Missing restaurantId or items | 400 | `{ "error": "restaurantId and at least one item are required" }` |
| Restaurant not found | 404 | `{ "error": "Restaurant not found" }` |
| Restaurant is closed | 400 | `{ "error": "Restaurant is currently closed" }` |
| A menuItemId not found or not from this restaurant | 400 | `{ "error": "One or more items are invalid for this restaurant" }` |
| Item is unavailable or insufficient stock | 400 | `{ "error": "Insufficient stock for one or more items" }` |

---

#### `GET /orders`

List the authenticated user's own orders. Behavior depends on role:

- **Customer:** orders they placed
- **Restaurant:** orders placed at restaurants they own
- **Rider:** orders currently assigned to them

**Query params:**

| Param | Description |
|---|---|
| `sort_by` | `asc` or `desc` (default sort field: `createdAt`) |
| `filter_field` | One of: `id`, `customerId`, `restaurantId`, `riderId`, `status`, `subtotal`, `deliveryFee`, `platformFee`, `total`, `cancelReason`, `createdAt`, `updatedAt` |
| `filter_value` | Exact value for `filter_field` |

**Examples:**

```http
GET /orders?sort_by=asc
GET /orders?filter_field=status&filter_value=delivered
GET /orders?filter_field=total&filter_value=600&sort_by=desc
```

**Success — 200:** Returns an array of order objects (with nested `items`).

**Failures:**

| Condition | Status | Response |
|---|---|---|
| No token | 401 | `{ "error": "Missing token" }` |
| Invalid list query params | 400 | e.g. `{ "error": "filter_value is required when filter_field is provided" }` |

---

#### `GET /orders/:id`

Get a single order by ID.

**Success — 200:** Returns the full order object with nested `items` and nested `timeline` (chronological, oldest first).

**Failures:**

| Condition | Status | Response |
|---|---|---|
| No token | 401 | `{ "error": "Missing token" }` |
| Order not found | 404 | `{ "error": "Order not found" }` |
| Requester's `id` is not the order's `customerId`, the owning restaurant's `ownerId`, or the order's `riderId` | 403 | `{ "error": "You do not have permission to view this order" }` |

---

#### `GET /orders/:id/timeline`

Get just the status timeline for an order — useful for tracking an order's progress without re-fetching the full order object. Same visibility rule as `GET /orders/:id`.

**Query params:**

| Param | Description |
|---|---|
| `sort_by` | `asc` or `desc` (default sort field: `occurredAt`) |
| `filter_field` | One of: `id`, `orderId`, `status`, `actorId`, `actorRole`, `occurredAt` |
| `filter_value` | Exact value for `filter_field` |

**Examples:**

```http
GET /orders/:id/timeline?sort_by=desc
GET /orders/:id/timeline?filter_field=status&filter_value=claimed
GET /orders/:id/timeline?filter_field=actorRole&filter_value=restaurant&sort_by=asc
```

**Success — 200:**
```json
[
  { "status": "placed", "actorId": "a1b2c3-...", "actorRole": "customer", "occurredAt": "2026-01-01T00:00:00.000Z" },
  { "status": "accepted", "actorId": "r1r2r3-...", "actorRole": "restaurant", "occurredAt": "2026-01-01T00:05:00.000Z" },
  { "status": "preparing", "actorId": "r1r2r3-...", "actorRole": "restaurant", "occurredAt": "2026-01-01T00:06:00.000Z" },
  { "status": "ready_for_pickup", "actorId": "r1r2r3-...", "actorRole": "restaurant", "occurredAt": "2026-01-01T00:20:00.000Z" },
  { "status": "claimed", "actorId": "k1k2k3-...", "actorRole": "rider", "occurredAt": "2026-01-01T00:22:00.000Z" },
  { "status": "picked_up", "actorId": "k1k2k3-...", "actorRole": "rider", "occurredAt": "2026-01-01T00:25:00.000Z" },
  { "status": "delivered", "actorId": "k1k2k3-...", "actorRole": "rider", "occurredAt": "2026-01-01T00:40:00.000Z" }
]
```
> Events are always in chronological order (oldest first). A `cancelled` order's timeline ends with a `cancelled` event instead of continuing further.

**Failures:**

| Condition | Status | Response |
|---|---|---|
| No token | 401 | `{ "error": "Missing token" }` |
| Order not found | 404 | `{ "error": "Order not found" }` |
| Requester's `id` is not the order's `customerId`, the owning restaurant's `ownerId`, or the order's `riderId` | 403 | `{ "error": "You do not have permission to view this order" }` |
| Invalid list query params | 400 | e.g. `{ "error": "filter_field is required when filter_value is provided" }` |

---

#### `GET /orders/available`

List orders that are `ready_for_pickup` and have no rider assigned yet. Rider role only.

**Query params:**

| Param | Description |
|---|---|
| `sort_by` | `asc` or `desc` (default sort field: `createdAt`) |
| `filter_field` | One of: `id`, `customerId`, `restaurantId`, `riderId`, `status`, `subtotal`, `deliveryFee`, `platformFee`, `total`, `cancelReason`, `createdAt`, `updatedAt` |
| `filter_value` | Exact value for `filter_field` |

**Examples:**

```http
GET /orders/available?sort_by=desc
GET /orders/available?filter_field=status&filter_value=ready_for_pickup
GET /orders/available?filter_field=deliveryFee&filter_value=50
```

**Success — 200:** Returns an array of order objects (without customer contact details).

**Failures:**

| Condition | Status | Response |
|---|---|---|
| No token | 401 | `{ "error": "Missing token" }` |
| Requester's role is not `rider` | 403 | `{ "error": "Only rider accounts can view available orders" }` |
| Invalid list query params | 400 | e.g. `{ "error": "filter_field must be one of: id, customerId, restaurantId, riderId, status, subtotal, deliveryFee, platformFee, total, cancelReason, createdAt, updatedAt" }` |

---

#### `PATCH /orders/:id/claim`

Claim an unassigned order for delivery. Rider role only.

**Success — 200:** Returns the updated order with `riderId` set to the requester and `status` unchanged (still `ready_for_pickup` until the rider marks it picked up). Records a `claimed` timeline event.

**Failures:**

| Condition | Status | Response |
|---|---|---|
| No token | 401 | `{ "error": "Missing token" }` |
| Requester's role is not `rider` | 403 | `{ "error": "Only rider accounts can claim orders" }` |
| Order not found | 404 | `{ "error": "Order not found" }` |
| Order is not `ready_for_pickup` | 400 | `{ "error": "Only orders ready for pickup can be claimed" }` |
| Order already has a rider assigned | 409 | `{ "error": "Order has already been claimed" }` |

---

#### `PATCH /orders/:id/status`

Advance an order's status by one step. Who is allowed depends on the transition — see the [Order Lifecycle](#order-lifecycle) table.

**Body:**
```json
{ "status": "accepted" }
```
> `status` must be the single next step in the lifecycle — you cannot skip steps (e.g. `placed` → `preparing` directly is rejected).

**Success — 200:** Returns the updated order object.

**Failures:**

| Condition | Status | Response |
|---|---|---|
| No token | 401 | `{ "error": "Missing token" }` |
| Order not found | 404 | `{ "error": "Order not found" }` |
| Missing status | 400 | `{ "error": "status is required" }` |
| Status is not a valid next step from the current status | 400 | `{ "error": "Cannot transition from <current> to <requested>" }` |
| Transition is restaurant-only and requester's `id` does not match the owning restaurant's `ownerId` | 403 | `{ "error": "Only the restaurant owner can perform this transition" }` |
| Transition is rider-only and requester's `id` does not match the order's `riderId` | 403 | `{ "error": "Only the assigned rider can perform this transition" }` |

---

#### `PATCH /orders/:id/cancel`

Cancel an order. Customer (order owner) only.

**Body:**
```json
{ "reason": "Changed my mind" }
```
> `reason` is optional.

**Success — 200 (cancelled from `placed`, free):**
```json
{
  "id": "o1o2o3-...",
  "status": "cancelled",
  "cancelReason": "Changed my mind",
  "cancellationFee": 0
}
```

**Success — 200 (cancelled from `accepted`, penalty applied):**
```json
{
  "id": "o1o2o3-...",
  "status": "cancelled",
  "cancelReason": "Changed my mind",
  "cancellationFee": 50
}
```
> The cancellation fee is deducted conceptually from a future refund; Panda-Lite does not model payments/refunds directly, it only reports the fee that would apply.

**Failures:**

| Condition | Status | Response |
|---|---|---|
| No token | 401 | `{ "error": "Missing token" }` |
| Order not found | 404 | `{ "error": "Order not found" }` |
| Requester is not the order's customer | 403 | `{ "error": "Only the customer who placed this order can cancel it" }` |
| Order status is `preparing` or later | 400 | `{ "error": "Orders can only be cancelled before preparation begins" }` |

---

### Ratings

#### `POST /orders/:id/rate`

Rate the restaurant and/or rider for a delivered order. Customer (order owner) only.

**Body:**
```json
{ "target": "restaurant", "score": 5, "comment": "Food was great and arrived hot!" }
```
> `target` must be `restaurant` or `rider`. `comment` is optional. `score` must be an integer 1–5.

**Success — 201:** Returns the new rating object.

**Failures:**

| Condition | Status | Response |
|---|---|---|
| No token | 401 | `{ "error": "Missing token" }` |
| Order not found | 404 | `{ "error": "Order not found" }` |
| Requester is not the order's customer | 403 | `{ "error": "Only the customer who placed this order can leave a rating" }` |
| Order is not `delivered` | 400 | `{ "error": "Only delivered orders can be rated" }` |
| Missing or invalid target | 400 | `{ "error": "target must be restaurant or rider" }` |
| Missing or out-of-range score | 400 | `{ "error": "score must be an integer between 1 and 5" }` |
| target is `rider` but order has no assigned rider | 400 | `{ "error": "This order has no rider to rate" }` |
| Already rated this target for this order | 409 | `{ "error": "You have already rated this order's <target>" }` |

---

#### `GET /restaurants/:id/ratings`

List all ratings left for a restaurant.

**Query params:**

| Param | Description |
|---|---|
| `sort_by` | `asc` or `desc` (default sort field: `createdAt`) |
| `filter_field` | One of: `id`, `orderId`, `customerId`, `target`, `targetId`, `score`, `createdAt` |
| `filter_value` | Exact value for `filter_field` |

**Examples:**

```http
GET /restaurants/:id/ratings?sort_by=asc
GET /restaurants/:id/ratings?filter_field=score&filter_value=5
GET /restaurants/:id/ratings?filter_field=target&filter_value=restaurant&sort_by=desc
```

**Success — 200:** Returns an array of rating objects where `target: "restaurant"`.

**Failures:**

| Condition | Status | Response |
|---|---|---|
| No token | 401 | `{ "error": "Missing token" }` |
| Restaurant not found | 404 | `{ "error": "Restaurant not found" }` |
| Invalid list query params | 400 | e.g. `{ "error": "filter_field must be one of: id, orderId, customerId, target, targetId, score, createdAt" }` |

---

## Permission Summary

| Action | Requirement |
|---|---|
| Create restaurant | Requester role must be `restaurant` |
| Update restaurant | Requester's `id` must equal the restaurant's `ownerId` — no other restaurant account, regardless of role, may update it |
| Manage menu (add/update items) | Requester's `id` must equal the owning restaurant's `ownerId` — no other restaurant account, regardless of role, may manage it |
| Place order | Requester role must be `customer` |
| Accept / prepare / mark ready | Requester's `id` must equal the owning restaurant's `ownerId` |
| Claim order | Requester role must be `rider`; order must be unclaimed |
| Mark picked up / delivered | Requester's `id` must equal the order's `riderId` |
| Cancel order | Requester's `id` must equal the order's `customerId`; only from `placed` or `accepted` |
| Rate order | Requester's `id` must equal the order's `customerId`; order must be `delivered` |
| View order (`GET /orders/:id`, `GET /orders/:id/timeline`) | Requester's `id` must equal exactly one of: the order's `customerId`, the owning restaurant's `ownerId`, or the order's `riderId`. No other user — including other customers, other restaurants, or other riders — may view it |

---

## Environment Variables

| Variable | Default | Description |
|---|---|---|
| `PORT` | `7000` | Port the server listens on |
| `APP_JWT_SECRET` | `dev_app_jwt_secret_change_me` | Secret used to sign app-level JWTs. **Change this in production.** |
| `CONTEXT_SIGNING_SECRET` | *(shared with gateway)* | Used to verify the signed `X-STQA-Context` header from the gateway |
| `TEAM_DATABASE_ADMIN_URL` | *(provided by gateway deployment)* | Admin connection string used to select each team's database |
| `DELIVERY_FEE` | `50` | Flat delivery fee applied to every order |
| `PLATFORM_FEE_PERCENT` | `10` | Platform fee as a percentage of `subtotal` |
| `CANCELLATION_PENALTY_PERCENT` | `100` of `deliveryFee` | Cancellation fee applied when cancelling from `accepted` (equal to `deliveryFee`) |

---
