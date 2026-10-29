# Open questions

Decided:
- Student `status`: active / inactive / banned (one field; registration tracked by `registered_at`).
- Buyers contact sellers via the seller's phone on the listing page.
- No editing of listings; seller can only remove.
- `.env` admin has only email + password.
- Admin enters email with the other student details; registration matches them and only sets the password.
- Phone is shown only to logged-in users.
- Search is `icontains` on title.
- Image limit is 5 photos, 5 MB each, enforced in the form.

Still open:
1. **Currency / pricing**: BDT only, negotiable flag?
2. **Deployment**: plaintext admin password in `.env` is fine locally; decide how it's provided if ever hosted.
