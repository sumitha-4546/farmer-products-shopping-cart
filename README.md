# 🌾 Farmer Products Shopping Cart System

A full-stack e-commerce application connecting farmers' products to customers — featuring a secure **Admin Portal** for product management and a **Customer Portal** for browsing, cart management, and checkout.

Built as a Full Stack Developer assessment project using **React (Vite)** on the frontend and **Python / FastAPI** on the backend.

![React](https://img.shields.io/badge/React-18.3-61DAFB?logo=react&logoColor=white)
![Vite](https://img.shields.io/badge/Vite-5.4-646CFF?logo=vite&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-0.110-009688?logo=fastapi&logoColor=white)
![Python](https://img.shields.io/badge/Python-3.10+-3776AB?logo=python&logoColor=white)
![SQLAlchemy](https://img.shields.io/badge/SQLAlchemy-ORM-D71F00)
![License](https://img.shields.io/badge/License-MIT-green)

---

## Table of Contents

- [Project Overview](#project-overview)
- [Features](#features)
- [Technology Stack](#technology-stack)
- [Project Structure](#project-structure)
- [Prerequisites](#prerequisites)
- [Backend Setup](#backend-setup)
- [Frontend Setup](#frontend-setup)
- [Database Setup](#database-setup)
- [Environment Variables](#environment-variables)
- [Steps to Run the Application](#steps-to-run-the-application-quick-reference)
- [API Reference](#api-reference)
- [Business Rules](#business-rules)
- [Assumptions Made During Development](#assumptions-made-during-development)
- [Screenshots](#screenshots)
- [License](#license)

---

## Project Overview

The **Farmer Products Shopping Cart System** allows farmers' products to be listed, managed, and sold directly to customers through a simple, responsive web application. It consists of two independent portals sharing one backend API:

- **Admin Portal** — secure login, add/edit/delete products, activate or deactivate listings, update stock levels, and view all products (active and inactive).
- **Customer Portal** — browse active products, search by name, filter by category, view product details, manage a shopping cart, and check out.

Business rules are enforced end-to-end on the server: only *Active* products are shown to customers, cart quantities can never exceed available stock, stock can never go negative, the grand total is calculated automatically, and stock is deducted from the database only after a successful checkout.

## Features

| Module   | Capabilities |
|----------|--------------|
| **Admin** | Secure JWT-based login · Create / edit / delete products · Activate / deactivate products · Update available stock · View all products |
| **Customer** | Browse active products · Search by name · Filter by category · View product details · Add to cart · Update / remove cart items · View cart with live totals · Checkout |

## Technology Stack

| Layer      | Technology |
|------------|------------|
| Frontend   | React.js (Vite), React Router, Axios, functional components + hooks |
| Backend    | Python, FastAPI, SQLAlchemy ORM, Pydantic |
| Auth       | JWT (python-jose) + bcrypt password hashing (passlib) |
| Database   | SQLite by default (zero-config) — swappable to PostgreSQL or MySQL via a single `.env` value |

## Project Structure

```
farmer-cart/
├── backend/
│   ├── app/
│   │   ├── main.py             # FastAPI app entrypoint, CORS, startup admin seeding
│   │   ├── database.py         # SQLAlchemy engine/session configuration
│   │   ├── models.py           # ORM models (Admin, Product, CartItem, Order, OrderItem)
│   │   ├── schemas.py          # Pydantic request/response schemas
│   │   ├── auth.py             # Password hashing + JWT helpers
│   │   └── routers/
│   │       ├── auth.py         # POST /api/auth/login
│   │       ├── products.py     # Product CRUD, activate/deactivate, stock update
│   │       ├── cart.py         # Add / update / remove / view cart
│   │       └── orders.py       # Checkout + view orders
│   ├── migrations/
│   │   └── schema.sql          # Plain SQL schema (reference / manual setup)
│   ├── requirements.txt
│   └── .env.example
└── frontend/
    ├── src/
    │   ├── pages/admin/         # Login, ProductList, AddProduct, EditProduct
    │   ├── pages/customer/      # ProductListing, ProductDetails, Cart, Checkout
    │   ├── components/          # Navbar, RequireAdmin, ProductForm
    │   ├── api/axios.js         # Axios instance + JWT interceptor + customer ID header
    │   └── App.jsx              # Route definitions
    ├── package.json
    └── .env.example
```

## Prerequisites

- **Python** 3.10+
- **Node.js** 18+ and npm
- *(Optional)* PostgreSQL or MySQL — only required if you don't want to use the default SQLite database

## Backend Setup

```bash
cd backend
python -m venv venv

# Activate the virtual environment
source venv/bin/activate        # macOS/Linux
venv\Scripts\activate           # Windows

pip install -r requirements.txt

# Create your local environment file
cp .env.example .env            # macOS/Linux
copy .env.example .env          # Windows
```

By default, `DATABASE_URL` in `.env` points to SQLite, so **no database installation is required** to run the app. All tables are created automatically on first startup via SQLAlchemy, and a default admin account is seeded from `ADMIN_USERNAME` / `ADMIN_PASSWORD` in `.env`.

Run the backend:

```bash
uvicorn app.main:app --reload --port 8000
```

- API base URL: `http://localhost:8000/api`
- Interactive API docs (Swagger UI): `http://localhost:8000/docs`
- Default admin login: **username `admin`, password `Admin@123`** *(change these in `.env` before first run if you want different credentials)*

## Frontend Setup

Open a **new terminal**:

```bash
cd frontend
npm install

cp .env.example .env            # macOS/Linux
copy .env.example .env          # Windows
```

`frontend/.env` already points to `http://localhost:8000/api`, matching the backend default — no changes needed if you used the default port.

Run the frontend:

```bash
npm run dev
```

- App URL: `http://localhost:5173`
- Customer Portal: `http://localhost:5173/`
- Admin Portal: `http://localhost:5173/admin/login`

> **Note:** This project uses **Vite**, not Create React App — the dev command is `npm run dev`, not `npm start`.

## Database Setup

The app works out of the box with **SQLite** (a single `farmer_cart.db` file, created automatically on first run). To switch to PostgreSQL or MySQL instead:

1. Create an empty database, e.g. `CREATE DATABASE farmer_cart;`
2. In `backend/requirements.txt`, uncomment `psycopg2-binary` (PostgreSQL) or `pymysql` (MySQL), then re-run `pip install -r requirements.txt`.
3. In `backend/.env`, set `DATABASE_URL` accordingly:
   - PostgreSQL: `DATABASE_URL=postgresql://postgres:password@localhost:5432/farmer_cart`
   - MySQL: `DATABASE_URL=mysql+pymysql://root:password@localhost:3306/farmer_cart`
4. Restart the backend — SQLAlchemy creates all tables automatically. `backend/migrations/schema.sql` is also provided for reference or manual setup.

## Environment Variables

**`backend/.env`**

| Variable | Description | Default |
|---|---|---|
| `DATABASE_URL` | SQLAlchemy connection string | `sqlite:///./farmer_cart.db` |
| `SECRET_KEY` | Secret used to sign JWT tokens | *(set your own)* |
| `ALGORITHM` | JWT signing algorithm | `HS256` |
| `ACCESS_TOKEN_EXPIRE_MINUTES` | Admin session length | `120` |
| `ADMIN_USERNAME` | Default admin username (auto-seeded) | `admin` |
| `ADMIN_PASSWORD` | Default admin password (auto-seeded) | `Admin@123` |
| `FRONTEND_ORIGIN` | Allowed CORS origin for the frontend | `http://localhost:5173` |

**`frontend/.env`**

| Variable | Description | Default |
|---|---|---|
| `VITE_API_BASE_URL` | Base URL the frontend calls | `http://localhost:8000/api` |

## Steps to Run the Application (Quick Reference)

```bash
# Terminal 1 — backend
cd backend
python -m venv venv && source venv/bin/activate   # Windows: venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env
uvicorn app.main:app --reload --port 8000

# Terminal 2 — frontend
cd frontend
npm install
cp .env.example .env
npm run dev
```

1. Open `http://localhost:5173` in your browser.
2. Shop as a customer immediately — no login needed.
3. Go to `http://localhost:5173/admin/login` and log in with `admin` / `Admin@123` to manage products.

## API Reference

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/auth/login` | Admin login, returns JWT |
| GET | `/api/products` | Public: list Active products (`?search=`, `?category=`) |
| GET | `/api/products/{id}` | Public: product details |
| GET | `/api/products/admin/all` | Admin: list all products (Active + Inactive) |
| POST | `/api/products/admin` | Admin: create product |
| PUT | `/api/products/admin/{id}` | Admin: update product |
| DELETE | `/api/products/admin/{id}` | Admin: delete product |
| PATCH | `/api/products/admin/{id}/status` | Admin: activate/deactivate |
| PATCH | `/api/products/admin/{id}/stock` | Admin: update stock |
| GET | `/api/cart/{customer_id}` | View cart |
| POST | `/api/cart/add` | Add item to cart |
| PUT | `/api/cart/update` | Update cart item quantity |
| DELETE | `/api/cart/remove/{cart_item_id}?customer_id=...` | Remove item from cart |
| POST | `/api/orders/checkout` | Checkout current cart |
| GET | `/api/orders/{customer_id}` | View a customer's past orders |

All admin endpoints require an `Authorization: Bearer <token>` header. Full interactive documentation is available at `/docs` while the backend is running.

## Business Rules

- Only products with status **Active** are visible to customers.
- Customers cannot add more of a product to their cart than is currently in stock.
- Product stock can never become negative.
- The grand total is calculated automatically and always reflects live product prices.
- Stock is deducted from the database only after a successful checkout.
- All forms (product create/edit, checkout) are validated on both the client and server.

## Assumptions Made During Development

1. **No customer registration/login** was specified in the requirements, so each browser is assigned a random persistent `customer_id` (stored in `localStorage`) to identify its cart and orders — this keeps the customer flow frictionless while still isolating each shopper's cart.
2. **Product images** are supported via an image URL field (the simplest, most portable option) rather than binary file upload, since the spec allowed "URL or Upload."
3. **SQLite is the default database** for zero-config local evaluation; the code is written with SQLAlchemy, so switching to PostgreSQL or MySQL is a one-line `.env` change (see [Database Setup](#database-setup)).
4. **Checkout is a single, all-or-nothing operation** — if any item in the cart no longer has enough stock at checkout time, the entire checkout is rejected with a clear error message and no stock is deducted.
5. Only **one admin role** exists (no multi-admin management UI), matching the spec's "Admin should be able to log in securely" requirement without requiring admin user management.
6. Deleting a product also removes any cart items referencing it (cascade), since a deleted product can no longer be purchased.

## Screenshots

> _Add screenshots of the Admin and Customer portals here, e.g.:_

| Customer — Product Listing | Customer — Cart | Admin — Product List |
|---|---|---|
| ![Product Listing](docs/screenshots/product-listing.png) | ![Cart](docs/screenshots/cart.png) | ![Admin Products](docs/screenshots/admin-products.png) |

## License

This project was built for evaluation purposes as part of a Full Stack Developer assessment. Feel free to reuse or adapt it for learning purposes.
