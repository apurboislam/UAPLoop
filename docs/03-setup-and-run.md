# 3. Setup and Run

## 3.1 What you need

- Python 3 installed
- A terminal (command line)

That is all. The database (SQLite) is built into Python.

## 3.2 The golden rule: use the virtual environment

A **virtual environment** (the `.venv/` folder) is a private box of Python packages just for this project. We never install packages for the whole computer. This keeps projects from breaking each other (for example, two projects needing different Django versions).

Every command uses `.venv/bin/python` or `.venv/bin/pip` so it runs inside the box.

## 3.3 First time setup

```bash
python3 -m venv .venv                        # 1. create the private box
.venv/bin/pip install -r requirements.txt    # 2. install Django, Pillow and friends
cp .env.example .env                         # 3. create your settings file
```

Now open `.env` and fill it in:

```
ADMIN_EMAIL=admin@uap.edu
ADMIN_PASSWORD=choose-a-strong-password
SECRET_KEY=any-long-random-text
DEBUG=1
```

| Setting | Meaning |
|---------|---------|
| `ADMIN_EMAIL`, `ADMIN_PASSWORD` | Login details of the admin account |
| `SECRET_KEY` | Django uses it to sign cookies and tokens. Keep it secret. |
| `DEBUG` | `1` shows detailed error pages (for development). `0` shows our normal 404 page. |

Then:

```bash
.venv/bin/python manage.py migrate          # 4. create the database tables
.venv/bin/python manage.py ensure_admin     # 5. create the admin account from .env
.venv/bin/python manage.py runserver        # 6. start the website
```

Open `http://127.0.0.1:8000/` for the site and `http://127.0.0.1:8000/admin/` for the admin panel.

## 3.4 What each command does

| Command | What happens |
|---------|--------------|
| `manage.py migrate` | Reads the migration files and creates or updates tables in `db.sqlite3` |
| `manage.py makemigrations` | After you change a model, creates a new migration file describing the change |
| `manage.py ensure_admin` | Our own command (in `accounts/management/commands/ensure_admin.py`). Creates the admin user or updates its password from `.env`. Safe to run many times. |
| `manage.py runserver` | Starts Django's development web server on port 8000 |
| `manage.py test` | Runs all automated tests |

## 3.5 Try it out (first use)

1. Log into `/admin/` with the admin email and password.
2. Add a **Category** (for example "Electronics").
3. Add a **User** (a student) with full name, university ID, phone, email, status Active. Do not set a password: the student will do that.
4. Log out, go to `/register/` and type the same university ID, phone, name and email, plus a new password.
5. You are now logged in as the student. Click **New listing** and post something.
6. Log in to `/admin/` again, open Listings, select the new one and run the **Approve** action.
7. The listing now appears on the home page.

## 3.6 Keeping requirements in sync

If you ever install a new package, update the list so others get it too:

```bash
.venv/bin/pip freeze > requirements.txt
```

Current `requirements.txt`:

| Package | Why |
|---------|-----|
| Django 6.1.2 | The framework |
| pillow 13.0.0 | Needed by Django's `ImageField` to check that an uploaded file is a real image |
| asgiref, sqlparse | Small helper libraries Django itself needs |
