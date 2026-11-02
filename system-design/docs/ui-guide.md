# UI guide: basic Bootstrap 5

Plain Django templates, Bootstrap 5 and a small local stylesheet. The app runs locally (`runserver` on localhost), no hosting, no build step.

## Rules
- Load the Bootstrap CSS file and the Bootstrap JavaScript bundle in `base.html`. The bundle is used for the collapsing navbar (hamburger menu on small screens).
- Load `static/css/styles.css` for the small background, heading, paragraph and highlight styles. The UAPLoop and UAP logos live in `static/images/`.
- Use Bootstrap classes for the grid, navigation, cards, buttons, forms, tables, badges and alerts. No icon library or extra frontend library is needed.
- Bootstrap CSS comes from a CDN.

## Base layout (`templates/base.html`, the only project-level template)
Page templates live inside their app: `accounts/templates/accounts/`, `catalog/templates/catalog/`.

- Dark navbar with the UAPLoop logo and Home link; logged-out users also see Login and a Register button. Logged-in users see My listings, Inbox, a New listing button and a Logout POST button.
- `container` with `{% block content %}`.
- Django messages shown as plain `alert` boxes (no close button).
- Dark footer with the UAP logo, an Explore menu, 74/A Green Road in Dhaka, +8802-41010626 and support@uaploop.com.

## Pages
| Page | What it has |
|---|---|
| Browse | Search form (title search, category select, type select), two to five cards per row with handover place and View item buttons, Clear filters or Create a listing when empty, Previous/Next buttons |
| Listing detail | Title, price, type badge, condition, category, handover place, description, photos, seller contact after a request, View requests / Chat with seller / Send request / Log in to request action as appropriate |
| Register | One form: university ID, phone, full name, email, password, confirm password |
| Login | One form: email, password |
| New listing | One form: category select, type select, title, description, required handover place, price, rent period, exchange for, condition, photos, Submit listing button. Note: "Your listing will appear after admin approval" |
| My listings | `table` with title, status `badge`, reject reason, View requests count and Remove buttons. No edit |
| Remove listing | Small page asking "Are you sure?" with a Yes button (instead of a modal) |
| Inbox | Conversations with Open chat buttons; empty state offers Browse listings |
| Chat | Messages with View item, Back to inbox and Send message buttons |

## Listing card
`card` with the cover photo on top, then title, price, type badge, handover place and a View item button. Plain text "No photo" when there is none.

## Status badges
pending: `text-bg-warning`, approved: `text-bg-success`, rejected: `text-bg-danger`.

## Forms
Render with `{{ form.as_p }}` inside a `card`, errors shown by Django's default. Submit with `btn btn-primary`.

## Price display
`Tk 1200`, and `Tk 200 / day` for rent.

## Admin pages
Django admin's own look, out of scope here.
