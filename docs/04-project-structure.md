# 4. Project Structure

```
UAPLoop/
├── manage.py                 Entry point for every Django command
├── requirements.txt          List of Python packages to install
├── .env.example              Template for the .env settings file
├── .env                      Your private settings (not in git)
├── db.sqlite3                The database file (created by migrate, not in git)
├── media/                    Uploaded listing photos (not in git)
├── CLAUDE.md, .ai/rules.md   Coding rules for the project
├── system-design/docs/       Planning notes written before coding
├── docs/                     This beginner documentation
│
├── config/                   Project settings (the "brain")
│   ├── settings.py           All configuration
│   ├── urls.py               Main URL list, includes each app's URLs
│   ├── forms.py              add_bootstrap() helper shared by all forms
│   ├── wsgi.py, asgi.py      Hooks for running on a real web server (Django default)
│
├── templates/                Shared templates
│   ├── base.html             Main layout: navbar, messages, footer
│   ├── 401.html              "Login required" page
│   └── 404.html              "Page not found" page
│
├── accounts/                 App 1: users and login
├── catalog/                  App 2: categories, listings, photos, buy requests
├── inbox/                    App 3: chat messages
└── reviews/                  App 4: reviews
```

## 4.1 Why split into apps?

Each app takes care of **one topic**. This makes the code easier to find and explain: anything about chat is in `inbox/`, anything about reviews is in `reviews/`. It is like chapters in a book.

The apps depend on each other in one direction only:

```
accounts  <--  catalog  <--  inbox
                  ^
                  └-------  reviews
```

`catalog` uses the user from `accounts`. `inbox` and `reviews` both use `catalog` (buy requests and listings).

## 4.2 Files inside every app

Every app follows the same layout, so once you understand one you understand all.

| File | Job |
|------|-----|
| `models.py` | Database tables for this app |
| `views.py` | Functions that handle requests |
| `urls.py` | Which URL calls which view |
| `forms.py` | Forms and their validation rules |
| `admin.py` | How the models appear in the admin panel |
| `templates/<app>/` | HTML pages for this app |
| `tests.py` | Automated tests |
| `migrations/` | Auto-generated database change files |
| `apps.py`, `__init__.py` | Small files Django needs to recognise the folder as an app |

Why are templates inside `templates/<app>/` (for example `catalog/templates/catalog/browse.html`)? Because Django searches all apps' `templates` folders together. The extra folder name stops two apps' files with the same name (like `form.html`) from clashing.

## 4.3 The `config/` folder in detail

### `settings.py`

| Setting | Value | Why |
|---------|-------|-----|
| `read_env()` | small function | Reads `.env` line by line into a dictionary, so secrets are not written in code. We wrote it ourselves instead of adding a package. |
| `SECRET_KEY` | from `.env` | Signs sessions and CSRF tokens |
| `DEBUG` | from `.env` | Developer error pages on or off |
| `INSTALLED_APPS` | Django apps + our 4 apps | Tells Django which apps exist |
| `MIDDLEWARE` | Django defaults | Sessions, CSRF protection, login, messages, clickjacking protection |
| `TEMPLATES['DIRS']` | `templates/` | Where shared templates live |
| `DATABASES` | SQLite, `db.sqlite3` | The database |
| `AUTH_USER_MODEL` | `accounts.User` | Use our own user model instead of Django's default |
| `LOGIN_URL` | `unauthorized` | When a page needs login, send the visitor to our 401 page |
| `LOGIN_REDIRECT_URL`, `LOGOUT_REDIRECT_URL` | `browse` | Go to the home page after login or logout |
| `TIME_ZONE` | `Asia/Dhaka` | Show times in Bangladesh time |
| `MEDIA_URL`, `MEDIA_ROOT` | `media/` | Where uploaded photos are saved and how they are served |

### `urls.py`

```python
urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('accounts.urls')),
    path('', include('catalog.urls')),
    path('', include('reviews.urls')),
    path('', include('inbox.urls')),
] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

It adds the admin site, then plugs in each app's URL list. The last line lets the development server show uploaded photos.

### `forms.py`

```python
def add_bootstrap(form):
    for field in form.fields.values():
        if isinstance(field.widget, forms.Select):
            field.widget.attrs['class'] = 'form-select'
        else:
            field.widget.attrs['class'] = 'form-control'
```

Bootstrap needs the CSS class `form-control` on inputs and `form-select` on dropdowns. Instead of writing this in every form, each form calls `add_bootstrap(self)` once. This is the only shared helper in the project.

## 4.4 Complete URL map

| URL | View | Login? | What it does |
|-----|------|--------|--------------|
| `/` | `catalog.browse` | No | Home page: approved listings, search, filters |
| `/listing/<id>/` | `catalog.detail` | No | One listing with photos and reviews |
| `/listing/<id>/buy/` | `catalog.request_buy` | Yes | POST: send a buy request |
| `/listing/<id>/requests/` | `catalog.listing_requests` | Yes (seller) | See who requested your item |
| `/request/<id>/accept/` | `catalog.accept_request` | Yes (seller) | POST: accept a request, item becomes sold |
| `/new/` | `catalog.new_listing` | Yes | Create a listing |
| `/mine/` | `catalog.my_listings` | Yes | Your listings and their status |
| `/sold/<id>/` | `catalog.mark_sold` | Yes (seller) | POST: mark sold without a request |
| `/remove/<id>/` | `catalog.remove_listing` | Yes (seller) | Confirm page, then POST deletes |
| `/listing/<id>/review/` | `reviews.add_review` | Yes (verified buyer) | POST: add a review |
| `/inbox/` | `inbox.inbox` | Yes | All your conversations |
| `/inbox/<id>/` | `inbox.conversation` | Yes (buyer or seller) | One chat |
| `/inbox/<id>/new/` | `inbox.new_messages` | Yes (buyer or seller) | JSON: messages newer than a given id (used by live chat) |
| `/register/` | `accounts.register_view` | No | Claim your student record |
| `/login/` | `accounts.login_view` | No | Log in |
| `/logout/` | `accounts.logout_view` | No | POST: log out |
| `/unauthorized/` | `accounts.unauthorized` | No | The 401 page |
| `/admin/` | Django admin | Staff | Admin panel |
| `/media/...` | Django static helper | No | Uploaded photos |
