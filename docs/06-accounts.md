# 6. Accounts: Users, Registration and Login

Folder: `accounts/`

## 6.1 The big idea: no open signup

On most websites anyone can sign up. UAPLoop is for UAP students only, so:

1. The **admin creates** a student record first (name, university ID, phone, email, status).
2. The student goes to `/register/` and types the **same details** plus a new password.
3. If everything matches, the student "claims" the record and sets a password.

So "registration" does **not create** a user. It **activates** one that already exists.

```
Admin creates record          Student registers              Student logs in
(no password yet)   ───────▶  (details must match)  ───────▶ (email + password)
registered_at = empty         password set,
                              registered_at = now
```

## 6.2 Registration (`RegisterForm` in `accounts/forms.py`)

The form has 6 fields: university ID, phone, full name, email, password, confirm password.

The `clean()` method is where the checking happens:

```python
def clean(self):
    data = super().clean()
    if self.errors:
        return data
    if data['password'] != data['confirm_password']:
        raise forms.ValidationError('Passwords do not match.')
    user = User.objects.filter(university_id=data['university_id'].strip()).first()
    if (
        user is None
        or user.status != 'active'
        or user.phone != only_digits(data['phone'])
        or (user.email or '').lower() != data['email'].lower()
        or user.full_name.lower() != data['full_name'].strip().lower()
    ):
        raise forms.ValidationError("Your details don't match.")
    if user.registered_at is not None:
        raise forms.ValidationError('This account is already registered. Please login.')
    self.user = user
    return data
```

Step by step:

1. If a field is already invalid (for example a bad email), stop.
2. The two passwords must be the same.
3. Find the student by university ID.
4. Check **all** of these: the record exists, it is active, the phone matches (digits only, so dashes and spaces do not matter), the email matches (ignoring capital letters), the name matches (ignoring capital letters and extra spaces).
5. If anything is wrong, show one general message: "Your details don't match."
6. If the student already registered, tell them to log in.

**Why one general error message?** If we said "phone is wrong", an attacker could guess details one at a time. A single message gives no hints.

**Why must the status be active?** Inactive or banned students must not be able to register.

`save()` then sets the password (Django **hashes** it, so the real password is never stored) and fills `registered_at` with the current time.

The view (`register_view`) logs the student in right away and sends them to the home page with "Welcome to UAPLoop."

## 6.3 Login (`LoginForm` and `login_view`)

`LoginForm` is Django's built-in `AuthenticationForm` with two changes:

- The username field is renamed and shown as an **Email** field.
- Friendlier error messages.

Django checks the email and password against the hashed password. It also rejects users whose `is_active` is False. Since our `User.save()` sets `is_active` from `status`, **inactive and banned users cannot log in**, with no extra code.

### The `next` parameter

If you open a page that needs login (like `/new/`) while logged out, you go to the 401 page with `?next=/new/`. After you log in, you are sent back to `/new/`.

```python
next_url = request.GET.get('next', '')
if url_has_allowed_host_and_scheme(next_url, allowed_hosts=request.get_host()):
    return redirect(next_url)
return redirect('browse')
```

`url_has_allowed_host_and_scheme` makes sure `next` points to **our own site**. Without it, an attacker could send a link like `/login/?next=http://evil.com/` and after login the student would land on a fake site. This is called an **open redirect** attack. A test proves it is blocked.

## 6.4 Logout

```python
def logout_view(request):
    if request.method == 'POST':
        logout(request)
    return redirect('browse')
```

Logout only works with POST (the navbar uses a small form with a button). Why? If logout worked with GET, any website could log you out just by including an image with `src="/logout/"`.

## 6.5 The 401 page

In settings, `LOGIN_URL = 'unauthorized'`. Django's `@login_required` sends logged-out visitors to `/unauthorized/?next=...`. Our `unauthorized` view shows `401.html` with status code 401 and a Login button that keeps the `next` value. This is clearer than silently jumping to the login form.

## 6.6 The admin account (`ensure_admin` command)

File: `accounts/management/commands/ensure_admin.py`

```bash
.venv/bin/python manage.py ensure_admin
```

It reads `ADMIN_EMAIL` and `ADMIN_PASSWORD` from `.env`, then creates that user (or updates it if it exists) with `is_staff=True`, `is_superuser=True`, status active, and the password.

**Why not Django's `createsuperuser`?** It asks questions in the terminal. Our command is automatic and can be run again any time, for example to reset a forgotten admin password by editing `.env`.

## 6.7 Admin side for users (`accounts/admin.py`)

- List shows name, university ID, phone, status, email, registration time.
- Filter by status, search by name, ID or phone.
- When the admin **creates** a user, the code sets an **unusable password**, so nobody can log in to that record until the real student registers.
- Action **"Reset registration"**: clears `registered_at` and the password. Use it if a student forgot their password or someone else claimed their record. The student can then register again.
