# 11. Templates and UI

## 11.1 One layout for all pages: `templates/base.html`

Every page starts with `{% extends 'base.html' %}` and fills `{% block content %}`. So the navbar, message alerts and footer are written **once**.

`base.html` contains:

- Bootstrap CSS and JS loaded from a CDN (jsDelivr), plus the small local `static/css/styles.css` file.
- A dark navbar. Links change based on `user.is_authenticated`:
  - Logged out: Home, Login, Register
  - Logged in: Home, My listings, Inbox, New listing, Logout (a small POST form)
- The navbar brand shows `static/images/uaploop-logo.svg` beside the UAPLoop name; both lead to the browse page.
- On small screens the navbar collapses into a "hamburger" button (Bootstrap's JS does this).
- A loop over `messages` that shows alerts like "Listing submitted...". These come from `messages.success(...)` in views.
- A dark footer that stays at the bottom (`d-flex flex-column min-vh-100` on body, `mt-auto` on footer). It shows `static/images/uap-logo.png`, UAPLoop's description, an Explore menu, 74/A Green Road in Dhaka, +8802-41010626 and support@uaploop.com.

## 11.2 Why only basic Bootstrap?

Bootstrap supplies the layout and components. The local stylesheet only adds simple background, heading, paragraph and highlight styles.

## 11.3 Responsive design (works on phones)

| Trick | Where | Effect |
|-------|-------|--------|
| `<meta name="viewport" ...>` | base.html | Phones show the real width instead of a zoomed-out desktop page |
| `row-cols-2 row-cols-md-3 row-cols-lg-5` | browse.html | 2 cards per row on phones, 3 on medium screens and 5 on large screens |
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
| `catalog/browse.html` | browse | Search and filters, cards with handover place and View item buttons, empty state links, Previous/Next |
| `catalog/detail.html` | detail | Photos, handover place, request box with action buttons, reviews |
| `catalog/new.html` | new_listing | Listing form with required handover place, photo input, Submit listing button |
| `catalog/mine.html` | my_listings | Table of own listings with View requests and Remove buttons |
| `catalog/requests.html` | listing_requests | Table of buy requests with Open chat and Accept buttons |
| `catalog/remove.html` | remove_listing | Confirmation |
| `inbox/inbox.html` | inbox | Table of conversations with Open chat buttons and Browse listings empty state |
| `inbox/conversation.html` | conversation | Messages, View item and Back to inbox buttons, Send message form, live update script |
| `reviews/section.html` | included in detail | Review form and list |

## 11.5 Useful template features we use

- `{{ form.as_p }}`: Django prints all form fields with labels and errors. Combined with `add_bootstrap()`, they look like Bootstrap inputs.
- `{{ listing.get_listing_type_display }}`: shows "Sell" instead of the stored value "sell".
- `{{ listing.price_text }}`: templates can call model methods without brackets.
- `{{ amount|floatformat:"-2" }}`: shows `500` instead of `500.00`, but keeps `499.50`.
- `{{ description|linebreaksbr }}`: keeps the line breaks the seller typed.
- `{% csrf_token %}`: in every POST form, required for security.
- Django **auto-escapes** every `{{ }}` value, so HTML typed by users is shown as text, not run.
