# 11. Templates and UI

## 11.1 One layout for all pages: `templates/base.html`

Every page starts with `{% extends 'base.html' %}` and fills `{% block content %}`. So the navbar, message alerts and footer are written **once**.

`base.html` contains:

- Bootstrap CSS and JS loaded from a CDN (jsDelivr). No CSS files of our own.
- A dark navbar. Links change based on `user.is_authenticated`:
  - Logged out: Browse, Login, Register
  - Logged in: Browse, New listing, My listings, Inbox, Logout (a small POST form)
- On small screens the navbar collapses into a "hamburger" button (Bootstrap's JS does this).
- A loop over `messages` that shows alerts like "Listing submitted...". These come from `messages.success(...)` in views.
- A footer that stays at the bottom (`d-flex flex-column min-vh-100` on body, `mt-auto` on footer).

## 11.2 Why only basic Bootstrap?

Project rule: no custom styling. Bootstrap gives a clean, consistent look and works on every screen size with just class names. It keeps the project simple to explain.

## 11.3 Responsive design (works on phones)

| Trick | Where | Effect |
|-------|-------|--------|
| `<meta name="viewport" ...>` | base.html | Phones show the real width instead of a zoomed-out desktop page |
| `row-cols-2 row-cols-sm-3 row-cols-md-4 row-cols-lg-6` | browse.html | 2 cards per row on phones, up to 6 on large screens |
| `col-md-8 col-lg-6` | forms | Forms are full width on phones, centred and narrower on big screens |
| `col-lg-7` / `col-lg-5` | detail.html | Photos and info side by side on big screens, stacked on phones |
| `table-responsive` | tables | Tables scroll sideways on small screens instead of breaking the page |
| `ratio ratio-1x1` + `object-fit-cover` | photo cards | Every photo is a neat square, no stretching |
| `text-break` | titles, messages | Very long words wrap instead of overflowing |

## 11.4 Page list

| Template | Used by | Content |
|----------|---------|---------|
| `base.html` | all | Layout |
| `401.html` | `unauthorized` | Login required, with Login button keeping `next` |
| `404.html` | Django | Page not found (shown when `DEBUG=0`) |
| `accounts/form.html` | register, login | One generic form page; the view passes `title` and `button` text. Reused for two pages. |
| `catalog/browse.html` | browse | Filter bar, card grid, Previous/Next |
| `catalog/detail.html` | detail | Photos, info, request box, reviews |
| `catalog/new.html` | new_listing | Listing form plus photo input |
| `catalog/mine.html` | my_listings | Table of own listings |
| `catalog/requests.html` | listing_requests | Table of buy requests |
| `catalog/remove.html` | remove_listing | Confirmation |
| `inbox/inbox.html` | inbox | Table of conversations |
| `inbox/conversation.html` | conversation | Messages, send form, live update script |
| `reviews/section.html` | included in detail | Review form and list |

## 11.5 Useful template features we use

- `{{ form.as_p }}`: Django prints all form fields with labels and errors. Combined with `add_bootstrap()`, they look like Bootstrap inputs.
- `{{ listing.get_listing_type_display }}`: shows "Sell" instead of the stored value "sell".
- `{{ listing.price_text }}`: templates can call model methods without brackets.
- `{{ amount|floatformat:"-2" }}`: shows `500` instead of `500.00`, but keeps `499.50`.
- `{{ description|linebreaksbr }}`: keeps the line breaks the seller typed.
- `{% csrf_token %}`: in every POST form, required for security.
- Django **auto-escapes** every `{{ }}` value, so HTML typed by users is shown as text, not run.
