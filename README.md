# MediStock

MediStock is a local Flask and SQLite pharmacy storefront and operations prototype. The customer website, staff order workspace and admin portal share one database. The project is designed for local review before a production database or hosting setup is chosen.

## Current portals

- **Customer website:** registration, sign-in, approved catalogue, search, categories, product details, cart, checkout, order history, payment-reference submission and paid receipts.
- **Staff portal:** shared order board, atomic order claims, payment-reference review, order processing, delivery status and cancellation of unpaid orders.
- **Admin portal:** product and category management, stock movements, staff and customer accounts, deliveries, checkout settings, inventory-import review, reports, exports and audit history.

All portals use the same SQLAlchemy models. Product names and prices are copied into order items at checkout, so later catalogue edits do not rewrite old orders. Stock is reserved at checkout, remains reserved while the order is unpaid, is deducted once after staff verify payment, and is released when staff cancel an unpaid order.

## Important business and pharmacy settings

The confirmed business details in the project brief are configured as MediStock, 3rd Parklands, Jambo Plaza, Nairobi, and M-Pesa Business Number `0737395204`. Payment instructions use **Send Money**. The site does not describe this number as a PayBill or Till and does not collect an M-Pesa PIN.

Checkout remains unavailable until an administrator confirms the tax setting and configures delivery fees. No tax rate or delivery price is invented. Only Nairobi is enabled as a starter delivery zone. Prescription-required products cannot be ordered in this preview because prescription submission and pharmacist review are not yet connected. A designated pharmacist account can approve product information before publication. Do not use this local preview for real sales, dispensing or patient-care decisions.

The supplied workbook is included at `data/source/Book1.xlsx`. It has no headers and contains rows where a number may be in either column. Setup stages the original cell values for admin review; it does not guess prices or quantities or publish products automatically. The supplied logo is cropped for use in the website header and portal.

## Technology

- Python 3.10 or newer
- Flask
- Flask-SQLAlchemy and SQLAlchemy
- Flask-Migrate / Alembic migrations
- Flask-Login, Flask-WTF CSRF protection, Flask-Limiter
- SQLite for local preview; SQLAlchemy can be configured for another database later
- HTML, CSS and JavaScript

## Folder structure

```text
MediStock_Ultimate/
├── medistock/
│   ├── routes/              # customer, staff, admin and authentication routes
│   ├── templates/           # separate portal templates
│   ├── models.py            # shared database models
│   ├── services.py          # money, audit and notification helpers
│   └── security.py          # server-side role checks
├── migrations/              # Alembic migration history
├── static/                  # CSS, JavaScript, logo and product placeholders
├── data/source/Book1.xlsx   # supplied inventory source workbook
├── instance/                # local database and runtime storage
├── run.py
├── requirements.txt
└── .env.example
```

## Install on Windows

1. Extract the project to a normal folder on the USB, for example `D:\MediStock_Ultimate`.
2. Open that folder in VS Code.
3. Run `setup_windows.bat`, or use these commands in the VS Code terminal:

```powershell
py -m venv .venv
.\.venv\Scripts\python.exe -m pip install --upgrade pip
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

If the `py` launcher is unavailable, use `python -m venv .venv` for the first command. The setup script creates a local `.env` with a random secret, applies migrations, adds the confirmed business details and starter categories, stages the workbook rows for review, and securely prompts for the first admin account. It does not ship an admin password.

To select the project interpreter in VS Code, press `Ctrl+Shift+P`, choose **Python: Select Interpreter**, and select `.venv\Scripts\python.exe`.

## Run locally

Run `run_windows.bat`, or in the VS Code terminal:

```powershell
.\.venv\Scripts\python.exe -m flask --app run.py db upgrade
.\.venv\Scripts\python.exe -m flask --app run.py init-business-data
.\.venv\Scripts\python.exe run.py
```

Open <http://127.0.0.1:5000>. Keep the terminal open while using the website.

If you skipped admin creation during setup, run:

```powershell
.\.venv\Scripts\python.exe -m flask --app run.py create-admin
```

The command prompts for the administrator’s name, email and password. It also asks whether that account is a designated pharmacist reviewer. Admin access is at `/admin`; staff access is at `/staff`; customer sign-in is at `/login`.

## First admin setup

1. Sign in at `/admin`.
2. Open **Business settings**. Confirm the tax treatment and rate with the appropriate professional, and enter approved Nairobi delivery fees. Checkout stays paused until these values are explicitly configured.
3. Open **Inventory import** to review the staged spreadsheet rows. No row is treated as a price or stock quantity without a human decision.
4. Create products with confirmed names, details, prices and stock. Add an approved product image when available.
5. Ask a designated pharmacist account to approve catalogue information before enabling the product for customers.
6. Create staff accounts. Staff sign in separately at `/staff/login`.

Product creation, price changes, stock adjustments, payment reviews, order changes, account changes and assignments are recorded in the audit log. Products are deactivated instead of deleted so old orders keep their product snapshots.

## Order and payment workflow

1. A customer registers, browses approved products and submits a delivery order.
2. The server checks current availability and reserves stock in a database transaction. The customer cannot submit a browser-calculated total.
3. The order stays open while unpaid; it does not expire automatically. Submitting an M-Pesa reference moves it into payment review. Staff must cancel an unpaid order to close it.
4. Customers pay by Send Money to the configured MediStock Business Number, then enter the transaction reference. A reference or screenshot alone is not proof of payment.
5. A staff member claims the order and checks the reference against official business records. The staff member records the verified amount and review note.
6. A successful verification marks the order paid, records who reviewed it and deducts reserved stock exactly once. A rejected reference leaves the order unpaid and its stock reserved until staff cancel the order.
7. Staff progress the paid order through processing, ready for dispatch, out for delivery and delivered. Admin can assign or reassign the handling staff and rider.
8. Unpaid cancellation releases the reservation. Refunds are recorded by an administrator; stock is restored only when returned items are explicitly marked suitable for resale.

No card, insurance or automated M-Pesa integration is present. Do not ask customers for their M-Pesa PIN. Email and SMS notifications are also not configured.

## Database and migrations

The local database is `instance/medistock.db`. Apply migrations with:

```powershell
.\.venv\Scripts\python.exe -m flask --app run.py db upgrade
```

After a model change, create and inspect a migration before applying it:

```powershell
.\.venv\Scripts\python.exe -m flask --app run.py db migrate -m "Describe the change"
.\.venv\Scripts\python.exe -m flask --app run.py db upgrade
```

Never remove the local database to apply a schema change. Back up `instance/medistock.db` first. For a consistent SQLite backup, stop the app before copying the database file.

## Security and production preparation

Customer passwords are hashed. Admin and staff routes check account roles on the server. All form posts use CSRF protection, login routes are rate limited, SQL uses SQLAlchemy, and sessions do not use a remember-me cookie. Signing out clears the session. Admin sign-in uses a browser-session cookie, so signing out or closing the whole browser normally requires another sign-in. Closing only the admin tab may leave the browser session active, and browser settings that restore session cookies can affect browser-session behavior.

Before public hosting, configure a strong secret in `.env`, HTTPS, persistent rate-limit storage, backups, an email provider for password reset, a production database, privacy and terms text reviewed for Kenya, and the pharmacy and data-protection workflows. The local development server is bound to `127.0.0.1` and is not a production server.

## Known gaps in this local preview

- Customer self-service password reset needs a verified email/SMS provider.
- Prescription upload and pharmacist review for prescription-required orders are not enabled; those items are blocked from checkout.
- The M-Pesa process is manual and does not connect to Safaricom transaction APIs.
- Tax and delivery fees must be entered and confirmed by the business before checkout.
- Privacy, terms, retention and production compliance content need professional review before launch.
- The project still needs broader workflow and security test coverage before it can be considered complete or production-ready.
