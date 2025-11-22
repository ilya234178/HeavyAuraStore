## Copilot / AI agent instructions for Heavyaura

This file is a concise, actionable orientation for AI coding agents working on this Django-based storefront.

Purpose
- Small Django app (apps: `main`, `cart`) using SQLite and local media. Make minimal, well-scoped changes and preserve existing URL namespaces and template structure.

Big picture (what to know fast)
- Django project at the repo root: `manage.py` and `heavyaura/settings.py`.
- Two primary apps: `main` (products, categories, pages) and `cart` (session-backed shopping cart).
- Templates live under each app (`main/templates/main/...`, `cart/templates/cart/...`) and `TEMPLATES['APP_DIRS']` is True.
- The cart is stored in the session under key `settings.CART_SESSION_ID` (value: `cart`).

Important files to reference when changing behavior
- `heavyaura/settings.py` — DB (SQLite), MEDIA_ROOT, CART_SESSION_ID, DEBUG flag.
- `main/models.py` — Product and Category models. Note: `Product.sell_price()` holds discount logic.
- `main/views.py`, `main/urls.py` — product list/detail and pagination patterns.
- `cart/cart.py` — session-backed Cart class used throughout the app.
- `cart/views.py`, `cart/forms.py`, `cart/urls.py` — add/remove flows and form expectations (POST + `CartAddProductForm`).

Run / dev commands (Windows PowerShell)
- Create/activate a venv and install dependencies (Django, Pillow for ImageField). Example (manual):
  - python -m venv .venv; .\.venv\Scripts\Activate.ps1; pip install django pillow
- Run migrations and dev server:
  - python manage.py makemigrations; python manage.py migrate
  - python manage.py runserver
- Tests: `python manage.py test` (standard Django test runner).

Project-specific patterns & conventions
- URL namespaces are used: `main` and `cart`. Use `reverse('main:product_detail', args=[slug])` and keep names stable.
- Models define `get_absolute_url()` for link generation; prefer using them when rendering product links.
- Templates expect `cart` in the context (provided by `cart.context_processors.cart`). If you change context processors, verify templates still render.
- Image uploads: `Product.image` uses `upload_to='products/%Y/%m/%d'` and `MEDIA_ROOT` is configured — keep file paths consistent.
- Forms: quantity selection uses `CartAddProductForm` with `TypedChoiceField` and a hidden `override` boolean; cart add/remove endpoints expect a POST and validated form.

Data flow (quick walkthrough)
- Product listing (main.views.product_list) → product detail → user submits quantity form → `cart.views.cart_add` (POST) → `cart.Cart.add()` stores item as
  {product_id: {'quantity': int, 'price': str(product.price)}} in the session.
- `cart.Cart.__iter__` reconstructs Product objects via `Product.objects.filter(id__in=product_ids)` and yields items with `price` (Decimal) and `total_price`.

Safe editing guidelines for AI
- Preserve session key `settings.CART_SESSION_ID`. Changing it requires adjustments in `settings.py`, templates, and context processor usage.
- Keep URL names and namespaces stable (they're referenced in templates and redirects).
- When modifying cart pricing logic, prefer updating `Product.sell_price()` and reading its result where pricing is displayed. The session stores the original `price` string; rely on model methods for derived prices.

Known issues to watch for (observed in code)
- `cart/cart.py` contains bugs that affect correctness:
  - `__len__` sums `item['quanttity']` (misspelled) — this returns 0 or fails; fix to `item['quantity']` when making changes.
  - `get_total_price` computes ((price - price) * discount / 100) which is always zero. Expected logic likely uses product.discount to compute reduced price (or call `product.sell_price()`).
If you touch this file, add tests and run `python manage.py test`.

Example edits the project expects from Copilot
- Add a new field to `Product` (e.g., SKU): update model, create migration, update admin and templates.
- Fix cart total computation: update `cart.Cart.get_total_price()` to use `product.sell_price()` or correct discount arithmetic, add unit tests for cart totals.

When in doubt
- Run the development server and exercise the flows: list → detail → add to cart → view cart. Media files are served when `DEBUG=True`.
- Keep changes small and testable; prefer adding unit tests for behavioral changes.

Request feedback: If any behavior or convention is missing from this file (for example, a CI workflow, commit message format, or additional third-party services), tell me where to look or provide sample files and I'll update the instructions.
