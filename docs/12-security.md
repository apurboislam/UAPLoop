# 12. Security

Even a small student project handles real names, phone numbers and passwords. Here is every protection and where it lives.

| Threat | What it means | How UAPLoop prevents it | Where |
|--------|---------------|-------------------------|-------|
| Fake accounts | Outsiders joining | Admin-created records only, registration must match all details | `accounts/forms.py` |
| Guessing details | Trying one field at a time | Single generic error "Your details don't match." | `RegisterForm.clean` |
| Password theft from database | Someone copies `db.sqlite3` | Passwords are **hashed** (PBKDF2 by Django), never stored in plain text | Django `set_password` |
| Banned users | Banned student keeps using the app | `status` controls `is_active`; Django refuses inactive logins | `User.save` |
| Seeing hidden listings | Guessing the id of a pending listing | All public queries use `.public()`, others get 404 | `ApprovedQuerySet` |
| Editing other people's things | `/remove/7/` on someone else's listing | Every owner action queries with `seller=request.user`, so it is 404 for others | catalog views |
| Reading other people's chats | Opening `/inbox/9/` | `my_conversations()` only includes chats where you are buyer or seller | inbox views |
| Fake reviews | Reviewing without buying | Must have an accepted request, checked on the server; admin approves | reviews views |
| CSRF | Another site submits a form as you | `{% csrf_token %}` in every form + `CsrfViewMiddleware` | all templates |
| Actions by link | A link that deletes or logs out | All changing actions accept POST only | all views |
| XSS | Injecting `<script>` in titles or messages | Template auto-escaping; chat JS uses `textContent` | templates, `conversation.html` |
| SQL injection | Typing SQL into search | Django ORM always uses parameters, we never build SQL strings | everywhere |
| Open redirect | `/login/?next=http://evil.com` | `url_has_allowed_host_and_scheme` | `login_view` |
| Malicious uploads | `.exe` renamed to `.jpg`, huge files | Pillow verifies the image, 5 MB and 5 photo limits | `ListingForm.clean` |
| File name tricks | Uploaded name like `../../x` | Files are renamed to a random UUID | `image_path` |
| Clickjacking | Our site hidden inside another site's frame | `XFrameOptionsMiddleware` | settings |
| Leaked secrets | Secret key in git | `SECRET_KEY` and admin password live in `.env`, which is in `.gitignore` | settings, `.gitignore` |
| Phone privacy | Phone visible to everyone | Seller phone shown only after a buy request; buyer phone only to that seller | `detail.html`, `requests.html` |

## Known gaps (honest list)

- No limit on how many times someone can try to register or log in (rate limiting). Planned, not built.
- `DEBUG=1` and `ALLOWED_HOSTS = []` are for local use only; a real server needs `DEBUG=0`, a host name, and HTTPS.
- The registration form does not run Django's password strength validators (length, common password). See [16-limitations-and-future.md](16-limitations-and-future.md).
