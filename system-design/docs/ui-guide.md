# UI guide: basic Bootstrap 5

Plain Django templates and the Bootstrap 5 stylesheet only. The app runs locally (`runserver` on localhost), no hosting, no build step.

## Rules
- Load only the Bootstrap CSS file (one `<link>` in `base.html`). No Bootstrap JavaScript, so no carousel, modal, dropdown, collapse or toast.
- No custom CSS file, no theme changes, no icon library, no extra frontend libraries.
- Use only basic classes: `container`, `row`, `col-*`, `navbar`, `card`, `btn`, `form-control`, `form-select`, `table`, `badge`, `alert`, `list-group`.
- CSS source: the Bootstrap CDN link is simplest. To work offline, save `bootstrap.min.css` into `static/` and link that instead.

## Base layout (`templates/base.html`, the only project-level template)
Page templates live inside their app: `accounts/templates/accounts/`, `catalog/templates/catalog/`.

- Simple `navbar` with links: Browse, Login, Register; when logged in: Browse, New listing, My listings, Logout (a small POST form button).
- `container` with `{% block content %}`.
- Django messages shown as plain `alert` boxes (no close button).

## Pages
| Page | What it has |
|---|---|
| Browse | Filter form (title search, category select, type select), listing cards in a `row` with `col-md-4`, simple next/previous links |
| Listing detail | Title, price, type badge, condition, category, description, photos stacked one after another, seller name and phone (logged-in users only) |
| Register | One form: university ID, phone, full name, email, password, confirm password |
| Login | One form: email, password |
| New listing | One form: category select, type select, title, description, price, rent period, exchange for, condition, photos. Note: "Your listing will appear after admin approval" |
| My listings | `table` with title, status `badge`, reject reason, Remove link. No edit |
| Remove listing | Small page asking "Are you sure?" with a Yes button (instead of a modal) |

## Listing card
`card` with the cover photo on top, then title, price and type badge. Plain text "No photo" when there is none.

## Status badges
pending: `text-bg-warning`, approved: `text-bg-success`, rejected: `text-bg-danger`.

## Forms
Render with `{{ form.as_p }}` inside a `card`, errors shown by Django's default. Submit with `btn btn-primary`.

## Price display
`Tk 1200`, and `Tk 200 / day` for rent.

## Admin pages
Django admin's own look, out of scope here.
