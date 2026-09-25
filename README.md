# Amazon-Style Shopping Backend

A complete FastAPI backend covering registration, login, product management,
cart, order placement, order history, and payment — built with a clean
folder structure, JWT auth, custom exceptions, and rotating file logging.

## Folder Structure

```
amazon_backend/
├── app/
│   ├── main.py               # FastAPI app, wiring, startup/shutdown
│   ├── config.py             # Settings (env-driven)
│   ├── database.py           # SQLAlchemy engine/session
│   ├── logger.py             # Rotating file + console logger
│   ├── models/                # SQLAlchemy ORM models
│   │   ├── user.py
│   │   ├── product.py
│   │   ├── cart.py
│   │   └── order.py           # Order, OrderItem, Payment
│   ├── schemas/                # Pydantic request/response schemas
│   ├── routers/                # API route handlers
│   │   ├── auth.py             # /api/auth/register, /login
│   │   ├── products.py         # /api/products (CRUD)
│   │   ├── cart.py             # /api/cart
│   │   ├── orders.py           # /api/orders
│   │   └── payment.py          # /api/payment
│   ├── auth/                   # Password hashing + JWT logic
│   └── exceptions/             # Custom exceptions + global handlers
├── seed_admin.py               # Creates a default admin user
├── requirements.txt
├── .env.example
└── README.md
```

## Setup

```bash
python -m venv venv
source venv/bin/activate        # venv\Scripts\activate on Windows
pip install -r requirements.txt

cp .env.example .env            # edit SECRET_KEY at minimum

python seed_admin.py            # creates admin@amazon.local / Admin@123

uvicorn app.main:app --reload
```

Visit **http://127.0.0.1:8000/docs** for interactive Swagger UI.

## Switching Database (SQL → NoSQL or another SQL engine)

The default is SQLite (zero setup, file-based). To switch:

- **PostgreSQL/MySQL**: just change `DATABASE_URL` in `.env` — no code changes needed,
  since SQLAlchemy abstracts the engine.
- **MongoDB (NoSQL)**: would require swapping `database.py`/`models/` for a
  Motor/PyMongo-based data layer, since SQLAlchemy is SQL-only. The
  routers and schemas are written so only the model/DB layer would need
  to change — routes call plain Python objects, not raw SQL.

## API Overview

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/auth/register` | POST | — | Register a new user |
| `/api/auth/login` | POST | — | Login, returns JWT |
| `/api/products/` | GET | — | List/search products |
| `/api/products/{id}` | GET | — | Fetch single product |
| `/api/products/` | POST | Admin | Add product |
| `/api/products/{id}` | PUT | Admin | Update product |
| `/api/products/{id}` | DELETE | Admin | Delete product |
| `/api/cart/` | GET | User | Get cart items |
| `/api/cart/` | POST | User | Add item to cart |
| `/api/cart/{product_id}` | PUT | User | Update quantity |
| `/api/cart/{product_id}` | DELETE | User | Remove item |
| `/api/orders/` | POST | User | Place order from cart |
| `/api/orders/` | GET | User | Fetch previous orders |
| `/api/orders/{id}` | GET | User | Fetch single order |
| `/api/payment/` | POST | User | Pay for an order (mock) |
| `/api/payment/{order_id}` | GET | User | Fetch payment for an order |

## Auth in Swagger UI

Click **Authorize**, use the email/password from `/api/auth/register`
(username field = your email). Or call `/api/auth/login` directly and
paste the `access_token` into the Authorize dialog as `Bearer <token>`.

## Notes

- Logs write to `logs/app.log` (rotating, 5MB × 5 backups) and console.
- All errors return `{"success": false, "error": "..."}` via global
  exception handlers in `app/exceptions/handlers.py` — no raw stack traces
  leak to the client.
- Order placement is transactional: stock is validated for every cart item
  before anything is written, and the whole operation rolls back on failure.
- Payment is mocked (no real gateway). Swap `routers/payment.py` for a real
  Razorpay/Stripe integration when needed — the rest of the app doesn't change.
