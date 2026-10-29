# UAPLoop Team Guide (4 people, 8+ commits each)

The team rebuilds UAPLoop step by step in a new GitHub repo. Every step tells you which files to copy from the finished project and gives you a commit message. You work one person at a time, like a relay race, so nobody gets merge conflicts.

| Person | Part | Commits |
|--------|------|---------|
| Person 1 | Project setup, Accounts (users, register, login), basic CSS | 11 |
| Person 2 | Catalog database and admin, Reviews app | 9 |
| Person 3 | Catalog pages (browse, detail, new, my listings, buy requests) | 11 |
| Person 4 | Inbox (chat), final layout, deploy | 9 |

Order: Person 1, then 2, then 3, then 4. When you finish, push and tell the next person.

The reference zip also contains `docs/`, `system-design/`, `.ai/` and `CLAUDE.md`. Do not copy those: they are not part of the team repo.

---

## Part A: One time setup

### A1. The owner prepares the reference copy

The owner (who has the finished project) runs this inside the finished project folder:

```
git archive --format=zip -o UAPLoop-reference.zip HEAD
```

Send `UAPLoop-reference.zip` to all 4 people. This is the "reference": the finished code you copy from.

### A2. The owner creates the team repo

1. On GitHub, create a new **empty** repo called `UAPLoop` (no README, no .gitignore).
2. Settings, Collaborators, add the other 3 people.

### A3. Every person does this once

```
mkdir uaploop-work
cd uaploop-work
unzip UAPLoop-reference.zip -d reference
git clone https://github.com/<owner>/UAPLoop.git
cd UAPLoop
git config user.name "Your Name"
git config user.email "the-email-of-your-github-account@example.com"
```

Your folders now look like this:

```
uaploop-work/
  reference/     finished project, only copy FROM here, never edit
  UAPLoop/       team repo, you work HERE
```

**The email must be the one on your GitHub account**, or GitHub will not count your commits.

All commands below run inside `uaploop-work/UAPLoop/`.

---

## Part B: How every step works

1. Copy the files the step lists from `../reference/` into the same path in `UAPLoop/`. Example:
   ```
   mkdir -p accounts
   cp ../reference/accounts/models.py accounts/models.py
   ```
   You can also copy with VS Code or a file manager. Same path, same file name.
2. If the step says **"only"** or **"add"**, do not copy the whole file. Open both files and copy just the part named.
3. Check that Django is happy:
   ```
   .venv/bin/python manage.py check
   ```
   You must see `System check identified no issues (0 silenced).`
4. Commit and push:
   ```
   git add .
   git commit -m "the message from the step"
   git push
   ```

**Windows:** use `.venv\Scripts\python` instead of `.venv/bin/python`, and `copy` instead of `cp`.

**Before your turn starts:** run `git pull`. If you are Person 2, 3 or 4, also run setup once:

```
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt
cp .env.example .env
.venv/bin/python manage.py migrate
```

**Normal errors during the relay:** some pages link to pages a later teammate builds. Opening those pages in the browser shows a `NoReverseMatch` error until that teammate is done. That is expected. After each commit only `manage.py check` must pass. Some tests also fail until later people finish; each person's last steps say when.

**When copying part of a views file:** copy the whole function, including the `@login_required` line above it if there is one, and leave one blank line between functions (two is the Python style used here).

---

## Person 1: Project setup, Accounts and CSS (11 commits)

### P1.1 Basic project files
Copy: `.gitignore`, `.python-version`, `requirements.txt`, `.env.example`

Then create the venv (it is ignored by git):
```
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt
```
Needs Python 3.12 or newer. No `check` yet (there is no Django project yet).

Commit: `Add gitignore, requirements and env example`

### P1.2 Django project
Copy: `manage.py`, `config/__init__.py`, `config/asgi.py`, `config/wsgi.py`, `config/settings.py`

Now edit `config/settings.py`:
- In `INSTALLED_APPS`, delete the 4 lines `'accounts',` `'catalog',` `'reviews',` `'inbox',`
- Delete the line `AUTH_USER_MODEL = 'accounts.User'`
- Delete the line `STATICFILES_DIRS = [BASE_DIR / 'static']` (it comes back in the CSS step)

Create `config/urls.py` with this (not the reference version yet):
```python
from django.conf import settings
from django.contrib import admin
from django.urls import include, path, re_path
from django.views.static import serve

urlpatterns = [
    path('admin/', admin.site.urls),
    re_path(r'^media/(?P<path>.*)$', serve, {'document_root': settings.MEDIA_ROOT}),
]
```

Run `check`. **Do not run `migrate` yet** (the custom user must exist first). If you did by mistake, delete `db.sqlite3`.

Commit: `Create Django project config`

### P1.3 Bootstrap form helper
Copy: `config/forms.py`

Commit: `Add helper that adds Bootstrap classes to forms`

### P1.4 Custom User model
Copy: `accounts/__init__.py`, `accounts/apps.py`, `accounts/models.py`, `accounts/migrations/__init__.py`, `accounts/migrations/0001_initial.py`

Edit `config/settings.py`:
- Add `'accounts',` back at the end of `INSTALLED_APPS`
- Add `AUTH_USER_MODEL = 'accounts.User'` back above `LOGIN_URL`

Run `check`, then `.venv/bin/python manage.py migrate`.

Commit: `Add custom User model with email login`

### P1.5 Users in the admin panel
Copy: `accounts/admin.py`

Commit: `Register users in Django admin with reset action`

### P1.6 ensure_admin command
Copy: `accounts/management/__init__.py`, `accounts/management/commands/__init__.py`, `accounts/management/commands/ensure_admin.py`

Test it: `cp .env.example .env`, put an email and password in `ADMIN_EMAIL` and `ADMIN_PASSWORD` in `.env`, then run `.venv/bin/python manage.py ensure_admin`. Do not commit `.env` (git ignores it).

Commit: `Add ensure_admin command to create the admin account`

### P1.7 Page layout and error pages
Copy: `templates/401.html`, `templates/404.html`

Create `templates/base.html` with this simpler version (Person 4 replaces it with the full one later):
```html
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>UAPLoop</title>
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body class="d-flex flex-column min-vh-100">
<nav class="navbar navbar-expand-lg navbar-dark bg-dark mb-4">
  <div class="container">
    <a class="navbar-brand" href="/">UAPLoop</a>
    <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#mainNav" aria-controls="mainNav" aria-expanded="false" aria-label="Toggle navigation">
      <span class="navbar-toggler-icon"></span>
    </button>
    <div class="collapse navbar-collapse" id="mainNav">
      <ul class="navbar-nav ms-auto">
        {% if user.is_authenticated %}
        <li class="nav-item">
          <form method="post" action="{% url 'logout' %}">
            {% csrf_token %}
            <button class="btn btn-link nav-link" type="submit">Logout</button>
          </form>
        </li>
        {% else %}
        <li class="nav-item"><a class="nav-link" href="{% url 'login' %}">Login</a></li>
        <li class="nav-item"><a class="nav-link" href="{% url 'register' %}">Register</a></li>
        {% endif %}
      </ul>
    </div>
  </div>
</nav>
<div class="container">
  {% for message in messages %}
  <div class="alert alert-info">{{ message }}</div>
  {% endfor %}
  {% block content %}{% endblock %}
</div>
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

Commit: `Add base layout and 401 and 404 pages`

### P1.8 Basic stylesheet
Copy: `static/css/styles.css`

Edit `config/settings.py`: add this line under `STATIC_ROOT = ...`:
```python
STATICFILES_DIRS = [BASE_DIR / 'static']
```

Edit `templates/base.html`:
- Add `{% load static %}` as the very first line (above `<!doctype html>`)
- Add this line right under the Bootstrap `<link ...>` line:
  ```html
  <link rel="stylesheet" type="text/css" href="{% static 'css/styles.css' %}">
  ```

Commit: `Add basic external CSS stylesheet`

### P1.9 Register and login forms
Copy: `accounts/forms.py`

Commit: `Add register form that claims an admin-created record`

### P1.10 Register, login and logout pages
Copy: `accounts/views.py`, `accounts/urls.py`, `accounts/templates/accounts/form.html`

Edit `config/urls.py`, add this line under the `admin/` line:
```python
    path('', include('accounts.urls')),
```
Run `check`. Run the server (`.venv/bin/python manage.py runserver`) and open http://127.0.0.1:8000/login/ to see the form. Logging in redirects to the browse page, which Person 3 builds, so an error after login is normal for now.

Commit: `Add register, login and logout pages`

### P1.11 Accounts tests
Copy: `accounts/tests.py`

These tests pass after Person 3 is done (they need the browse page).

Commit: `Add accounts tests`

Push. Tell Person 2.

---

## Person 2: Catalog database, admin and Reviews (9 commits)

Run `git pull` and the "before your turn" setup from Part B.

### P2.1 Catalog app
Copy: `catalog/__init__.py`, `catalog/apps.py`, `catalog/migrations/__init__.py`

Edit `config/settings.py`: add `'catalog',` under `'accounts',` in `INSTALLED_APPS`.

Commit: `Create catalog app`

### P2.2 Catalog models
Copy: `catalog/models.py`, `catalog/migrations/0001_initial.py`

Run `check`, then `.venv/bin/python manage.py migrate`.

Commit: `Add Category, Listing, ListingImage and BuyRequest models`

### P2.3 Catalog admin
Copy: `catalog/admin.py`

Test: run the server, log in at http://127.0.0.1:8000/admin/ with the admin from `.env` (run `ensure_admin` first), add a category.

Commit: `Add catalog admin with approve action`

### P2.4 Catalog forms
Copy: `catalog/forms.py`

Commit: `Add listing form with photo checks and buy request form`

### P2.5 Reviews app and model
Copy: `reviews/__init__.py`, `reviews/apps.py`, `reviews/models.py`, `reviews/migrations/__init__.py`, `reviews/migrations/0001_initial.py`

Edit `config/settings.py`: add `'reviews',` under `'catalog',`.

Run `check`, then `migrate`.

Commit: `Add Review model`

### P2.6 Reviews admin
Copy: `reviews/admin.py`

Commit: `Add reviews to admin for approval`

### P2.7 Review form
Copy: `reviews/forms.py`

Commit: `Add review form`

### P2.8 Add review view
Copy: `reviews/views.py`, `reviews/urls.py`, `reviews/templates/reviews/section.html`

Edit `config/urls.py`, add under the accounts line:
```python
    path('', include('reviews.urls')),
```

Commit: `Add review submit view and review section template`

### P2.9 Reviews tests
Copy: `reviews/tests.py`

These pass after Person 4 is done.

Commit: `Add reviews tests`

Push. Tell Person 3.

---

## Person 3: Catalog pages (11 commits)

Run `git pull` and the "before your turn" setup from Part B.

In this part you build `catalog/views.py` and `catalog/urls.py` **one function at a time**. Keep `../reference/catalog/views.py` and `../reference/catalog/urls.py` open next to yours.

### P3.1 Browse page
- Create `catalog/views.py` with **only** the import lines at the top of the reference file, plus the `browse` function.
- Create `catalog/urls.py` with the 4 top lines and only the `browse` path:
  ```python
  from django.urls import path

  from . import views

  urlpatterns = [
      path('', views.browse, name='browse'),
  ]
  ```
- Copy: `catalog/templates/catalog/browse.html`
- Edit `config/urls.py`, add under the accounts line (so it sits above reviews):
  ```python
      path('', include('catalog.urls')),
  ```

Commit: `Add browse page with search and filters`

### P3.2 Listing detail page
Add the `detail` function to `views.py` and its `path(...)` line to `urls.py`. Copy `catalog/templates/catalog/detail.html`.

Commit: `Add listing detail page`

### P3.3 New listing page
Add `new_listing` (with its `@login_required`) and its path. Copy `catalog/templates/catalog/new.html`.

Commit: `Add new listing page with photo upload`

### P3.4 My listings page
Add `my_listings` and its path. Copy `catalog/templates/catalog/mine.html`.

Commit: `Add my listings page`

### P3.5 Mark as sold
Add `mark_sold` and its path.

Commit: `Let seller mark a listing as sold`

### P3.6 Remove listing
Add `remove_listing` and its path. Copy `catalog/templates/catalog/remove.html`.

Commit: `Let seller remove a listing and its photos`

### P3.7 Buy request
Add `request_buy` and its path.

Commit: `Let buyers send a buy request`

### P3.8 Requests list for the seller
Add `listing_requests` and its path. Copy `catalog/templates/catalog/requests.html`.

Commit: `Show buy requests to the seller`

### P3.9 Accept request
Add `accept_request` and its path. Now your `views.py` and `urls.py` should match the reference exactly. Compare them.

Commit: `Let seller accept a buy request`

### P3.10 Navbar links
In `templates/base.html`, add these 3 lines right after `<ul class="navbar-nav ms-auto">`, and put the first line above `{% if user.is_authenticated %}`, the other two right below it:
```html
        <li class="nav-item"><a class="nav-link" href="{% url 'browse' %}">Browse</a></li>
        <li class="nav-item"><a class="nav-link" href="{% url 'new_listing' %}">New listing</a></li>
        <li class="nav-item"><a class="nav-link" href="{% url 'my_listings' %}">My listings</a></li>
```
Look at `../reference/templates/base.html` if unsure where they go.

Commit: `Add catalog links to the navbar`

### P3.11 Catalog tests
Copy: `catalog/tests.py`

Run `.venv/bin/python manage.py test accounts`. It should pass now.

Commit: `Add catalog tests`

Push. Tell Person 4.

---

## Person 4: Inbox, final layout, deploy (9 commits)

Run `git pull` and the "before your turn" setup from Part B.

### P4.1 Inbox app
Copy: `inbox/__init__.py`, `inbox/apps.py`, `inbox/migrations/__init__.py`

Edit `config/settings.py`: add `'inbox',` under `'reviews',`.

Commit: `Create inbox app`

### P4.2 Message model
Copy: `inbox/models.py`, `inbox/migrations/0001_initial.py`

Run `check`, then `migrate`.

Commit: `Add Message model linked to a buy request`

### P4.3 Message form
Copy: `inbox/forms.py`

Commit: `Add message form`

### P4.4 Inbox page
- Create `inbox/views.py` with the import lines plus **only** `my_conversations` and `inbox`.
- Create `inbox/urls.py` with only the `inbox/` path (same shape as in P3.1).
- Copy: `inbox/templates/inbox/inbox.html`
- Edit `config/urls.py`, add under the reviews line:
  ```python
      path('', include('inbox.urls')),
  ```
  Now `config/urls.py` should match the reference exactly.

Commit: `Add inbox page listing my chats`

### P4.5 Conversation page
Add the `conversation` function and its path. Copy `inbox/templates/inbox/conversation.html`.

Commit: `Add chat page for one buy request`

### P4.6 Live new messages
Add the `new_messages` function and its path. Now `inbox/views.py` and `inbox/urls.py` match the reference.

Commit: `Add endpoint that returns new chat messages`

### P4.7 Final layout
Copy `templates/base.html` from the reference (replace the whole file). This adds the Inbox link and the footer. The CSS lines from Person 1 are already in it.

Commit: `Add inbox link and footer to the layout`

### P4.8 Inbox tests, run everything
Copy: `inbox/tests.py`

Run all tests:
```
.venv/bin/python manage.py test
```
Everything must pass. Then click through the whole app in the browser:
1. In admin, add a category and two students (users without a password).
2. Register as student A, create a listing.
3. In admin, approve the listing.
4. Register as student B (another browser or private window), open the listing, send a buy request, chat with the seller.
5. As student A, accept the request. As student B, write a review. In admin, approve the review.

Commit: `Add inbox tests`

### P4.9 Deploy file
Copy: `Procfile`

Commit: `Add Procfile for DigitalOcean App Platform`

Last check, this should print nothing:
```
diff -r ../reference . -x .git -x .venv -x db.sqlite3 -x media -x .env -x __pycache__ -x docs -x system-design -x .ai -x CLAUDE.md -x TEAM-GUIDE.md
```

Push. Done.

---

## Check that everyone has 8+ commits

```
git shortlog -sn
```

If someone shows up twice with different names, their `user.email` was different on one computer. Fix it for next time with `git config user.email`.

## If something goes wrong

| Problem | Fix |
|---------|-----|
| `git push` rejected | `git pull --rebase`, then `git push` |
| `ModuleNotFoundError: No module named 'django'` | You used system Python. Use `.venv/bin/python` |
| `check` says app not found | You added the app to `INSTALLED_APPS` before copying its `__init__.py` and `apps.py` |
| `InconsistentMigrationHistory` or user table errors | Delete `db.sqlite3` and run `migrate` again |
| `NoReverseMatch` in the browser | A later teammate builds that page. Normal until the relay ends |
| `TemplateDoesNotExist` | The template is in the wrong folder. It must be `<app>/templates/<app>/file.html` |
