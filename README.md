# The Paper Project — Frontend

**Subject:** Web Technologies / Full Stack Development
**Semester:** II
**Submission Type:** End-Semester Project

---

## Project Overview

The Paper Project is a campus-based stationery e-commerce web application developed as part of the Semester II End-Semester Project. The frontend is a fully functional, multi-page web interface built using core web technologies — HTML5, CSS3, and vanilla JavaScript — with no external frameworks or build tools.

The application allows students to browse a curated catalogue of stationery products, add items to a cart, and place delivery orders directly to their hostel rooms. An admin interface is also provided for managing product listings.

---

## Objectives

- To design and implement a responsive, multi-page web application using HTML, CSS, and JavaScript.
- To demonstrate client-server communication using the `fetch` API and RESTful HTTP methods (GET, POST, PATCH, DELETE).
- To implement persistent client-side state management using the browser's `localStorage`.
- To integrate a frontend interface with a Python Flask backend connected to a Supabase (PostgreSQL) database.

---

## Tech Stack

| Layer | Technology Used |
|---|---|
| Markup | HTML5 |
| Styling | Vanilla CSS3 (CSS Custom Properties, CSS Grid, Flexbox) |
| Scripting | Vanilla JavaScript (ES6+) |
| Fonts | Custom TTF — `Self-Modern` (display) · `HelveticaCustom` (body) |
| State Management | Browser `localStorage` |
| API Communication | `fetch` API (asynchronous, JSON over HTTP) |
| Backend | Flask REST API (Python) — separate repository |
| Database | Supabase (PostgreSQL) — managed via backend |

---

## Project Structure

```
frontend-PT/
│
├── index.html          # Main storefront — product catalogue, cart, checkout flow
├── add-product.html    # Admin page — add a new product to the catalogue
├── edit-product.html   # Admin page — edit an existing product by ID
│
├── images/             # Local image assets (hero banner, featured sections, etc.)
│   ├── hero.jpeg
│   ├── feat1.jpeg  feat2.jpeg  feat3.jpeg
│   ├── desk1.jpeg  desk2.jpeg  desk3.jpeg
│   └── fold1.jpeg  fold2.jpeg  fold3.jpeg  fold4.jpeg
│
├── Self-Modern.ttf     # Custom display/serif font
└── Helvetica.ttf       # Custom sans-serif body font
```

> Each HTML page is self-contained. Styles and scripts are embedded directly within the file using `<style>` and `<script>` tags, requiring no external CSS files, JS modules, or a build/compilation step.

---

## System Architecture

The following diagram illustrates how the frontend interacts with the backend:

```
 ┌─────────────────────────────────────────────────────┐
 │                    BROWSER (Client)                 │
 │                                                     │
 │  index.html          add-product.html               │
 │  ├── Product Grid    └── POST /products             │
 │  ├── Cart Sidebar                                   │
 │  ├── Checkout Form   edit-product.html              │
 │  │   └── POST /checkout   ├── GET /products         │
 │  └── GET, DELETE /products└── PATCH /products/<id>  │
 └────────────────────┬────────────────────────────────┘
                      │  HTTP (fetch API · JSON)
                      ▼
 ┌────────────────────────────────────────────────────┐
 │           Flask REST API  (backend-PT)             │
 │           https://backend-pt-nr2g.onrender.com     │
 │                                                    │
 │   Routes: GET/POST /products                       │
 │           PATCH/DELETE /products/<id>              │
 │           POST /checkout                           │
 └────────────────────┬───────────────────────────────┘
                      │  supabase-py client
                      ▼
 ┌────────────────────────────────────────────────────┐
 │               Supabase (PostgreSQL)                │
 │   ┌──────────────────┐   ┌────────────────────┐    │
 │   │     products     │   │    order_table     │    │
 │   └──────────────────┘   └────────────────────┘    │
 └────────────────────────────────────────────────────┘
```

---

## Page-Wise Description

### 1. `index.html` — Main Storefront

This is the primary page that a customer interacts with. It is divided into the following sections:

| Section | Description |
|---|---|
| Announcement Bar | Displays a promotional message (free delivery threshold) |
| Sticky Header & Navigation | Logo, navigation links, live cart item count badge |
| Hero Section | Full-width editorial layout with a local image |
| Featured Row | 3-column editorial grid showcasing product categories |
| Personalization Split Section | 2-column image + text block for custom orders |
| Desk Details (Triple Grid) | 3-column editorial grid — stickers, pens, accessories |
| The Fold (Quad Grid) | 4-column editorial / blog-style content grid |
| Product Catalogue (`#root`) | Dynamic grid rendered from live API data |
| Promo Banner | Static discount code display |
| Cart Sidebar | Slide-in panel; items persisted in `localStorage` |
| Checkout Form | Delivery details form (name, room no., building no.) |
| Footer | 4-column informational link grid |

**Responsive Design:** A media query at `max-width: 768px` collapses multi-column grids to single-column layouts, expands the cart sidebar to full viewport width, and resizes the hero section for mobile screens.

---

### 2. `add-product.html` — Admin: Add Product

Provides a centered form (max-width 480px) for creating a new product listing. Fields:
- **Title** — product name
- **Price (₹)** — numeric, minimum ₹0.01
- **Rating** — float between 1.0 and 5.0
- **Image URL** — optional; backend uses a fallback image if omitted

On form submission, a `POST` request is sent to `/products` with a JSON payload. The page redirects to `index.html` on success.

---

### 3. `edit-product.html` — Admin: Edit Product

Reads a product ID from the URL query string (`?id=<n>`). On page load:

1. Sends a `GET /products` request to fetch all products.
2. Finds the matching product by ID and pre-fills the form.
3. On submission, sends a `PATCH /products/<id>` request with the updated fields.
4. Redirects to `index.html` on success.

If no `id` parameter is present in the URL, the page immediately redirects to `index.html`.

---

## JavaScript Functionality (`index.html`)

### Global Constants and State

```js
const API_URL      = "https://backend-pt-nr2g.onrender.com/products";
const CHECKOUT_URL = "https://backend-pt-nr2g.onrender.com/checkout";

let allProducts = [];                                    // populated from API
let cart = JSON.parse(localStorage.getItem('cart')) || []; // hydrated on load
```

### Function Reference

| Function | Purpose |
|---|---|
| `fetchProducts()` | Calls `GET /products`; stores response in `allProducts`; triggers render |
| `renderProducts(products)` | Generates and injects product card HTML into `#root` |
| `handleDeleteProduct(id)` | Confirms deletion → `DELETE /products/<id>` → reloads page |
| `addToCart(id)` | Locates product in `allProducts` → appends to `cart` → saves & re-renders |
| `removeFromCart(index)` | Removes item by index from `cart` → saves & re-renders |
| `saveCart()` | Serialises `cart` to `localStorage`; updates the cart count badge |
| `renderCart()` | Builds the cart sidebar HTML; calculates the running total |
| `toggleCart()` | Toggles `.open` class on `#cart-sidebar` and `#cart-overlay` |
| `showCheckoutForm()` | Hides the "Checkout" button; reveals `#checkout-section` |
| `handleCheckout(e)` | Collects delivery details → `POST /checkout` → clears cart on success |

### Cart Persistence

Cart items are serialised as a JSON array and stored in `localStorage` under the key `'cart'`. On every page load the cart is hydrated from `localStorage` before the API call completes, ensuring the count badge and sidebar contents are immediately correct.

---

## Backend Integration

The frontend communicates with a Flask REST API deployed on **Render**.

### Deployed API Base URL

```
https://backend-pt-nr2g.onrender.com
```

### API Endpoints Used

| Page | User Action | HTTP Method | Endpoint |
|---|---|---|---|
| `index.html` | Load product catalogue | `GET` | `/products` |
| `index.html` | Delete a product (admin) | `DELETE` | `/products/<id>` |
| `index.html` | Place an order (checkout) | `POST` | `/checkout` |
| `add-product.html` | Add a new product (admin) | `POST` | `/products` |
| `edit-product.html` | Pre-fill edit form | `GET` | `/products` |
| `edit-product.html` | Save product edits (admin) | `PATCH` | `/products/<id>` |

### Sample Payloads

**Checkout — `POST /checkout`**
```json
{
  "name": "Lakshya",
  "room_no": "204",
  "building_number": "B3",
  "order": [
    { "id": 1, "title": "Fountain Pen" }
  ],
  "payable_amount": 1299.00
}
```

**Add Product — `POST /products`**
```json
{
  "title": "Midnight Fountain Pen",
  "price": 1299.00,
  "rating": 4.5,
  "thumbnail": "https://example.com/pen.jpg"
}
```

### CORS

The Flask backend uses `flask-cors` with a global `CORS(app)` configuration, allowing the browser to make cross-origin requests from any origin during development.

---

## Design System

All styling is defined using CSS Custom Properties declared in the `:root` block of each page:

| Token | Value | Purpose |
|---|---|---|
| `--cream` | `#faf7f0` | Primary page background |
| `--cream-pink` | `#f6dfd1` | Announcement bar background |
| `--charcoal` | `#1e2525` | Header, primary buttons, footer |
| `--accent` | `#c9a96e` | Accent / highlight colour |
| `--border` | `#e0dbd5` | Input and card borders |
| `--font-primary` | `Self-Modern`, Georgia | Display headings and titles |
| `--font-secondary` | `HelveticaCustom`, Helvetica | Body text, labels, buttons |

---

## How to Run Locally

No installation or build step is required. The files can be served using any static file server.

```bash
# Option 1 — Using npx serve (recommended to avoid font loading issues)
npx -y serve .
# → Open http://localhost:3000 in the browser

# Option 2 — Open directly in browser (may have font CORS restrictions)
open index.html
```

### Connecting to a Local Backend

To point the frontend at a locally running Flask backend, replace the `API_URL` constants in all three HTML files:

```js
// Before (production)
const API_URL = "https://backend-pt-nr2g.onrender.com/products";

// After (local development)
const API_URL = "http://localhost:5001/products";
```

Refer to the [backend README](../../backend/backend-PT/README.md) for instructions on setting up and running the Flask API locally.

---

## Limitations and Future Scope

- Admin controls (Edit / Delete buttons) are visible to all users. A login/authentication layer should be added to restrict admin actions in production.
- The cart currently allows duplicate entries for the same product (no quantity aggregation).
- All styling is duplicated across the three HTML files; refactoring into a shared external stylesheet would improve maintainability.
- Product images rely on external URLs; a file upload mechanism would improve the admin workflow.
