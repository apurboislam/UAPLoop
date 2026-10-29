# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project rule #1: nothing installed system-wide

All Python tooling lives in the project venv at `.venv/` (never `pip install` globally, never use system Python for packages). Use `.venv/bin/python` / `.venv/bin/pip` directly, or `source .venv/bin/activate`. Keep `requirements.txt` in sync (`.venv/bin/pip freeze > requirements.txt`).

## Commands

- Setup from scratch: `python3 -m venv .venv && .venv/bin/pip install -r requirements.txt`
- Run server: `.venv/bin/python manage.py runserver`
- Migrate: `.venv/bin/python manage.py makemigrations && .venv/bin/python manage.py migrate`
- Tests: `.venv/bin/python manage.py test` (single: `.venv/bin/python manage.py test app.tests.TestClass.test_name`)

## Status

Basic v1 is built: `accounts` (custom User, register/login/logout, `ensure_admin` command), `reviews` (verified-buyer reviews with admin approval, shown on the listing detail page) `inbox` (one chat per buy request, between that buyer and the item's seller) and `catalog` (Category, Listing, ListingImage, browse/detail/new/mine/remove views, Django admin moderation). Each app owns its models, views, urls, forms, admin, templates (`<app>/templates/<app>/`) and tests. Only the shared layout `templates/base.html` lives at project level. Config comes from `.env` (copy `.env.example`). First run: `migrate`, then `ensure_admin`.

Not built yet (kept out on purpose): registration rate-limit, default `pending` filter in the listing admin, bulk Reject action with reason (set status and reason on the listing change page instead).

## Design docs

`system-design/docs/` holds the design, one concern per file (database, auth flow, admin panel, basic Bootstrap 5 UI guide, open questions). Read it before changing models or flows, and keep it in sync with the code.

## What this is

UAPLoop is a Django app for University of Asia Pacific (UAP) students to list, sell, exchange, and rent personal items (calculators, phones, badminton rackets, cricket bats: anything). Stack: Django, SQLite, local filesystem storage for item photos (`MEDIA_ROOT`, no cloud storage).

## Domain rules (cross-cutting; span models, auth, and views)

- **Admin-provisioned users only.** There is no open signup. Admin pre-creates a student record (phone, college/university ID, name, status: active, inactive or banned). The student then "registers" from the app by supplying those details; they must match an admin-created record exactly, and only then can they set a password. Registration = claiming a pre-existing record, not creating one.
- **Login is email + password** (after registration), even though the pre-registered identity is keyed by phone/college ID. Only active users can register or sign in (inactive and banned cannot).
- **Admin-managed categories.** Categories are created/edited by admin only; a listing must have exactly one category chosen from them at creation.
- **Moderation queue.** Every new listing starts in a pending state and is invisible in public listings until an admin approves it. Public/user-facing queries must filter on approved status: enforce this in one place (e.g. a custom manager/queryset) rather than per view.
- **Listing types:** sell, exchange, rent (a listing has a type).
- **Photos** are uploaded to local disk; serve via Django media settings in dev.

## Notes for implementation

- Custom user model needs to be decided before the first migration (email as `USERNAME_FIELD`, linked to/created from the admin-provisioned student record). Set `AUTH_USER_MODEL` before running any `migrate`.
- Use Django admin for the admin-side workflows (student provisioning, category management, listing approval) unless told otherwise.

## Code style

Strict rules live in `.ai/rules.md`: simple code only, no comments, no em dashes, basic Bootstrap only. Read it first.

This is a basic student project. Write simple, beginner-level Django: function-based views, plain forms/models, no clever abstractions, no decorators/metaclasses/typing tricks, and no code comments. Keep the code short and readable.
