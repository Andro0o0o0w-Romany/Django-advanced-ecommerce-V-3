# Django Advanced E-Commerce V3

A full-featured, production-ready e-commerce platform built with Django 4.2. Supports product catalog management, user authentication with email verification, session-based shopping cart, PayPal payment integration, and a powerful admin dashboard.

---

## Table of Contents

- [Features](#features)
- [Tech Stack](#tech-stack)
- [Architecture](#architecture)
- [Project Structure](#project-structure)
- [Models & Database Schema](#models--database-schema)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [API & URL Endpoints](#api--url-endpoints)
- [Screenshots](#screenshots)
- [License](#license)

---

## Features

### User Management
- Custom user model with **email-based authentication**
- User registration with **email verification** (activation link)
- Password reset via email with secure token validation
- Editable user profile (avatar, address, phone number)
- Automatic **session timeout** after inactivity (configurable)

### Product Catalog
- Category-based product organization with SEO-friendly slugs
- **Product gallery** supporting multiple images per product
- Full-text **search** across product names and descriptions
- **Review & rating system** (1-5 stars) with per-user enforcement
- Product attributes and inventory management with SKU tracking
- Real-time stock control

### Shopping Cart
- **Dual-mode cart**: session-based for guests, user-linked for authenticated users
- Seamless cart transfer on login (guest cart merges into user cart)
- Add, increment, decrement, and delete operations
- Live cart item counter across all pages via context processor
- Automatic **10% tax calculation**

### Order & Payment
- Complete checkout flow with shipping details
- **PayPal Standard** payment integration (IPN callbacks)
- Auto-generated order numbers (`YYYYMMDD` + order ID)
- Order confirmation **email notifications**
- Order history and detailed order view per user
- Stock auto-decrement on successful order
- Order status tracking: `New` > `Accepted` > `Completed` | `Cancelled`

### Admin Dashboard
- Custom admin interface with inline editing for products, orders, and users
- Product management with inline gallery uploads and image thumbnails
- Order management with payment tracking and status workflow
- User account administration with profile picture thumbnails

---

## Tech Stack

| Layer        | Technology                          |
|--------------|-------------------------------------|
| Backend      | Django 4.2, Python 3.11             |
| Database     | SQLite3 (swappable to PostgreSQL)   |
| Frontend     | Bootstrap 4, jQuery, FontAwesome 5  |
| Payments     | PayPal Standard IPN (django-paypal) |
| Auth         | Django AbstractBaseUser (custom)    |
| Email        | SMTP via Django EmailMessage        |
| Config       | python-decouple (.env)              |
| Sessions     | django-session-timeout              |
| Admin        | django-admin-thumbnails             |
| Images       | Pillow                              |

---

## Architecture

```
                    +-----------------------+
                    |   AdvancedEcommerce    |
                    |   (Project Config)     |
                    |   settings / urls      |
                    +-----------+-----------+
                                |
          +----------+----------+----------+----------+
          |          |          |          |          |
     +----v---+ +---v----+ +--v-----+ +-v-------+  |
     |Accounts| | store  | |  Cart  | | Orders  |  |
     |  App   | |  App   | |  App   | |   App   |  |
     +--------+ +--------+ +--------+ +---------+  |
          |          |          |          |         |
          |   +------v------+  |   +------v------+  |
          |   | Context     |  |   | PayPal IPN  |  |
          |   | Processors  |  |   | Integration |  |
          |   +-------------+  |   +-------------+  |
          |                    |                     |
     +----v--------------------v---------------------v----+
     |                   SQLite3 Database                  |
     +----------------------------------------------------+
```

### App Responsibilities

| App | Responsibility |
|-----|---------------|
| **AdvancedEcommerce** | Project settings, root URL routing, WSGI/ASGI config |
| **Accounts** | Custom user model, registration, login/logout, email verification, password reset, profile management |
| **store** | Product & category CRUD, product gallery, review/rating system, search, attribute & inventory models |
| **Cart** | Session & user-based cart, add/remove/delete operations, tax calculation, checkout page |
| **Orders** | Order creation, PayPal payment flow, order history, email notifications, stock management |

### Context Processors

| Processor | Location | Purpose |
|-----------|----------|---------|
| `menu_links` | `store/context_processors.py` | Injects category list into every template for navigation |
| `counter` | `Cart/context_processors.py` | Injects cart item count into every template for the navbar badge |

---

## Project Structure

```
Django-advanced-ecommerce-V-3/
├── AdvancedEcommerce/           # Project configuration
│   ├── settings.py              # Django settings (apps, middleware, DB, email, PayPal)
│   ├── urls.py                  # Root URL dispatcher
│   ├── wsgi.py / asgi.py        # Server entry points
│
├── Accounts/                    # Authentication & user management
│   ├── models.py                # Account (custom user), UserProfile
│   ├── views.py                 # Register, login, logout, activate, dashboard, profile
│   ├── forms.py                 # RegistrationForm, UserForm, UserProfileForm
│   ├── urls.py                  # /accounts/* routes
│   ├── admin.py                 # AccountAdmin, UserProfileAdmin
│   └── templates/Accounts/      # Login, register, dashboard, email templates
│
├── store/                       # Product catalog
│   ├── models.py                # Category, Product, ProductGallery, ReviewRating,
│   │                            #   Attribute, AttributeValue, Inventory, StockControl
│   ├── views.py                 # Home, store listing, product detail, search, reviews
│   ├── forms.py                 # ReviewForm
│   ├── context_processors.py    # Category menu links
│   ├── urls.py                  # /, /store/, /product/<id>, /search/, /<category_slug>
│   ├── admin.py                 # ProductAdmin (with gallery inline), CategoryAdmin
│   ├── static/store/            # CSS, JS, fonts, images
│   └── templates/store/         # Product pages, search results, base layout
│
├── Cart/                        # Shopping cart
│   ├── models.py                # Cart, CartItem
│   ├── views.py                 # add_cart, remove_cart, delete_cart, cart, checkout
│   ├── context_processors.py    # Cart item counter
│   ├── urls.py                  # /cart/* routes
│   └── templates/Cart/          # Checkout template
│
├── Orders/                      # Orders & payments
│   ├── models.py                # Order, Payment, OrderProduct
│   ├── views.py                 # place_order, order_complete, order_detail
│   ├── forms.py                 # OrderForm
│   ├── urls.py                  # /orders/* routes + PayPal IPN
│   ├── admin.py                 # OrderAdmin (with OrderProduct inline)
│   └── templates/Orders/        # Order placement, confirmation, detail, email
│
├── static/                      # Collected static files
├── media/                       # User uploads (product images, profile pictures)
├── db.sqlite3                   # SQLite database
├── manage.py                    # Django management CLI
├── store.json / backup.json     # Fixture data for seeding
└── LICENSE                      # Proprietary license
```

---

## Models & Database Schema

### Accounts

```
Account (AbstractBaseUser)
├── email (PK for auth - USERNAME_FIELD)
├── username, first_name, last_name
├── phonenumber
├── is_admin, is_staff, is_active, is_superadmin
└── data_joined, last_login

UserProfile
├── user → Account (OneToOne)
├── profile_picture
├── address_line_1, address_line_2
├── city, state, country
└── phone_number
```

### Store

```
Category
├── name, slug (unique)
└── is_active

Product
├── name, slug, description, image
├── category → Category (FK)
├── price, stock
└── is_active

ProductGallery
├── product → Product (FK)
└── image

ReviewRating
├── product → Product (FK)
├── user → Account (FK)
├── subject, review, rating (1-5)
└── created_at, updated_at

Attribute ──► AttributeValue
                 └── value, attribute (FK)

Inventory
├── sku (unique), is_active, is_default
├── product → AttributeValue (M2M)
└── StockControl (OneToOne)
       └── units, last_checked
```

### Cart

```
Cart
├── cart_id (session key)
└── date_added

CartItem
├── user → Account (FK, nullable)
├── product → Product (FK)
├── cart → Cart (FK)
├── quantity
└── is_active
```

### Orders

```
Payment
├── user → Account (FK)
├── payment_id, payment_method
├── amount_paid, status
└── created_at

Order
├── user → Account (FK)
├── payment → Payment (FK)
├── order_number (auto-generated)
├── Shipping: first_name, last_name, phone, email,
│             address_line_1/2, country, state, city
├── order_total, tax, status
├── is_ordered, ip
└── created_at, updated_at

OrderProduct
├── order → Order (FK)
├── payment → Payment (FK)
├── user → Account (FK)
├── product → Product (FK)
├── quantity, product_price
└── ordered (bool)
```

---

## Installation

### Prerequisites

- Python 3.11+
- pip
- virtualenv (recommended)

### Setup

```bash
# Clone the repository
git clone https://github.com/Andro0o0o0w-Romany/Django-advanced-ecommerce-V-3.git
cd Django-advanced-ecommerce-V-3

# Create and activate virtual environment
python -m venv venv
source venv/bin/activate        # Linux/macOS
venv\Scripts\activate           # Windows

# Install dependencies
pip install django==4.2.7
pip install python-decouple
pip install Pillow
pip install django-paypal
pip install django-session-timeout
pip install django-admin-thumbnails

# Create .env file (see Configuration section below)

# Run migrations
python manage.py migrate

# Load seed data (optional)
python manage.py loaddata store.json

# Create superuser
python manage.py createsuperuser

# Start the development server
python manage.py runserver
```

---

## Configuration

Create a `.env` file in the project root with the following variables:

```env
SECRET_KEY=your-django-secret-key
DEBUG=True

# Email (SMTP)
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_HOST_USER=your-email@gmail.com
EMAIL_HOST_PASSWORD=your-app-password
EMAIL_USE_TLS=True

# PayPal
PAYPAL_RECEIVER_EMAIL=your-paypal-email@example.com
```

### Key Settings

| Setting | Value | Description |
|---------|-------|-------------|
| `AUTH_USER_MODEL` | `Accounts.Account` | Custom user model with email auth |
| `SESSION_EXPIRE_SECONDS` | `3600` | Session timeout (1 hour) |
| `SESSION_EXPIRE_AFTER_LAST_ACTIVITY` | `True` | Reset timer on activity |
| `PAYPAL_TEST` | `True` | Use PayPal sandbox |
| `DEFAULT_AUTO_FIELD` | `BigAutoField` | Default primary key type |

---

## Usage

### User Flow

1. **Register** at `/accounts/register/` - receive activation email
2. **Activate** account via the email link
3. **Browse** products at `/` or `/store/`
4. **Search** products via the search bar
5. **Filter** by category from the navigation menu
6. **View** product details, gallery, and reviews at `/product/<id>`
7. **Add to cart** - works for both guests and logged-in users
8. **Checkout** at `/cart/checkout/` (requires login)
9. **Place order** with shipping details
10. **Pay** via PayPal
11. **View orders** in the dashboard at `/accounts/my_orders/`

### Admin Flow

1. Access admin at the custom admin URL
2. Manage products with inline image gallery
3. Track and update order statuses
4. View payment records
5. Manage user accounts and profiles

---

## API & URL Endpoints

### Store Routes

| URL | View | Description |
|-----|------|-------------|
| `/` | `home` | Homepage with featured products |
| `/store/` | `home_store` | Full product listing |
| `/<category_slug>/` | `store` | Products filtered by category (paginated) |
| `/product/<id>` | `product_detail` | Single product with gallery & reviews |
| `/search/` | `search` | Search results page |
| `/submit_review/<product_id>` | `submit_review` | Submit/update a product review |

### Account Routes

| URL | View | Description |
|-----|------|-------------|
| `/accounts/register/` | `register` | User registration |
| `/accounts/login/` | `login` | User login |
| `/accounts/logout/` | `logout` | User logout |
| `/accounts/activate/<uidb64>/<token>/` | `activate` | Email activation |
| `/accounts/dashboard/` | `dashboard` | User dashboard |
| `/accounts/my_orders/` | `my_orders` | Order history |
| `/accounts/edit_profile/` | `edit_profile` | Edit profile & address |
| `/accounts/change_password/` | `change_password` | Change password |
| `/accounts/forgot_password/` | `forgot_password` | Request password reset |
| `/accounts/reset_password/` | `reset_password` | Set new password |

### Cart Routes

| URL | View | Description |
|-----|------|-------------|
| `/cart/` | `cart` | View shopping cart |
| `/cart/add_cart/<product_id>` | `add_cart` | Add item to cart |
| `/cart/remove_cart/<product_id>/<cart_item_id>/` | `remove_cart` | Decrement or remove item |
| `/cart/delete_cart/<product_id>/<cart_item_id>/` | `delete_cart` | Delete entire cart item |
| `/cart/checkout/` | `checkout` | Checkout page |

### Order Routes

| URL | View | Description |
|-----|------|-------------|
| `/orders/place_order/` | `place_order` | Create order & pay |
| `/orders/order_complete/` | `order_complete` | Order confirmation |
| `/orders/order_detail/<order_id>/` | `order_detail` | View order details |
| `/orders/payment_unsuccessful/` | `payment_unsuccessful` | Payment failure page |
| `/orders/paypal/` | PayPal IPN | PayPal callback handler |

---

## License

This project is proprietary software. Copyright (c) 2026 Andrew Romany. All Rights Reserved.
See the [LICENSE](LICENSE) file for details.
