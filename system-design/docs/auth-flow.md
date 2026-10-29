# Auth flow

## Provisioning (admin)
Admin adds a student in Django admin: full_name, university_id, phone, email, status (`active` / `inactive` / `banned`). Row is created with an unusable password (`registered_at` is null, read-only in admin).

## Registration (student, from the app)
Form fields: university_id, phone, full_name, email, password (+confirm).
1. Look up User by `university_id`.
2. Reject with one generic error ("details don't match") if not found, phone/name/email don't match the admin entry, or status is not `active`: don't reveal which check failed.
3. Once all details match, `registered_at` already set gives "already registered, please login".
4. On match: `set_password`, `registered_at=now`. Registration only sets the password, it never changes the other fields.
5. Log the user in.

Phone is compared normalised (digits only). Not in v1: rate-limiting this endpoint (it is an ID-guessing oracle), add it before any hosting.

## Login
Email + password via Django `authenticate` (email as `USERNAME_FIELD`). `inactive` or `banned` ⇒ refused. One combined message is shown for wrong password and inactive account ("wrong email or password, or your account is not active, contact admin"). If admin bans a logged-in user, `is_active` goes false and the session stops working.

## Not in v1
Email verification, password reset by email (no mail server yet; admin uses the **Reset registration** action so the student registers again and sets a new password), social login.
