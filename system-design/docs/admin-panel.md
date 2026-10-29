# Admin panel

Use Django's built-in admin (`/admin/`). No custom admin UI.

## Credentials from `.env`
`.env` (git-ignored; commit a `.env.example` with blank values):

```
ADMIN_EMAIL=
ADMIN_PASSWORD=
SECRET_KEY=
DEBUG=1
```

- The admin account holds only email + password (no name, ID or phone).
- Loaded in `config/settings.py` by a small hand-written parser (no extra library). `SECRET_KEY` and `DEBUG` come from there too.
- A management command `ensure_admin` creates/updates the superuser from `ADMIN_EMAIL`/`ADMIN_PASSWORD` (idempotent, run after `migrate`). Password lives in `.env` only; the DB stores the hash.
- Changing `.env` + re-running the command rotates the password.

## What admin does
| Task | Admin screen |
|---|---|
| Add / ban students | `User`: enter full_name, university_id, phone, email, status. List filter: status; search: name, university_id, phone. `email` and `registered_at` are filled by admin and the registration flow: `registered_at` is read-only |
| Let a student register again | `User` action **Reset registration**: clears the password and `registered_at`, keeps the email |
| Manage categories | `Category` |
| Moderate listings | `Listing`: filter by status; bulk action **Approve**; to reject, open the listing, set status to rejected and fill `reject_reason`; `ListingImage` as inline so photos are visible while reviewing |

Approve, and any status change saved on the listing page, set `reviewed_by` and `reviewed_at`.

Not in v1: default `pending` filter, bulk Reject with a reason prompt.
