# 10. Admin Panel

URL: `/admin/`. Log in with the email and password from `.env` (after running `ensure_admin`).

## 10.1 Why use Django admin instead of building our own?

Django generates a complete, secure management site from our models: lists, search, filters, add/edit forms, permissions and login. Building the same by hand would take many views and templates. For an admin-only tool, it is the perfect fit.

We only customised it a little in each app's `admin.py`.

## 10.2 Admin tasks and how to do them

| Task | Where | How |
|------|-------|-----|
| Add a student | Users > Add | Fill full name, university ID, phone, email, status Active. Leave password alone (it is set unusable automatically). |
| Block a student | Users > open student | Set status to Inactive or Banned. They can no longer log in or register. |
| Let a student register again | Users > select > action "Reset registration" | Clears password and registration time |
| Add a category | Categories > Add | Name (slug fills itself), active, sort order |
| Hide a category | Categories > open | Untick "is active" |
| Approve listings | Listings > filter by Pending > select > action "Approve" | Many at once |
| Reject a listing | Listings > open | Status Rejected, write a reject reason, save |
| See photos of a listing | Listings > open | Photos are shown inside (inline) |
| Approve reviews | Reviews > select > "Approve reviews" | |
| View buy requests | Buy requests | Read only use |

## 10.3 Customisations explained

| Code | What it does |
|------|--------------|
| `list_display` | Columns in the list page |
| `list_filter` | Filter box on the right side |
| `search_fields` | Search box |
| `actions` | Bulk actions in the dropdown (approve, reset registration) |
| `inlines` | Show related rows (listing photos) inside the edit page |
| `prepopulated_fields` | Fill slug from name automatically |
| `readonly_fields` | `registered_at` cannot be edited by hand |
| `save_model` | Extra code that runs when admin saves (set unusable password for new users; record who reviewed a listing) |

## 10.4 Note about bulk actions

The bulk actions use `queryset.update(...)`, which writes to the database directly in one query. It is fast. It skips the model's `save()` method, which is fine here because approving only changes the status fields.
