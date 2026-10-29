# 16. Limitations and Future Work

Every project has limits. Knowing them shows you understand your own code.

## 16.1 Known limitations

| Limitation | Why it is like this | How to improve |
|------------|---------------------|----------------|
| No rate limit on login or registration | Kept out on purpose for v1 | Count failed attempts per IP or account and block for a few minutes |
| Registration does not run password strength checks | `RegisterForm` sets the password without calling `validate_password`, so the validators in settings are not applied there | Call `django.contrib.auth.password_validation.validate_password` in `RegisterForm.clean` |
| No password reset by email | No email server in a local project | Use Django's built-in password reset views with an SMTP server |
| Sellers cannot edit listings | Simpler and safer (no change after approval) | Add an edit view that sets the status back to pending |
| Seller could accept more than one request by sending POSTs by hand | The Accept button hides after the item is sold, but `accept_request` does not check `is_sold` itself | Add `and not buy_request.listing.is_sold` to the check in `accept_request` |
| Chat uses polling every 0.5 s | No extra libraries or servers needed | Use WebSockets (Django Channels) for large numbers of users |
| No notifications for new messages | Out of scope for v1 | Show an unread count in the navbar |
| Admin listing page does not filter "pending" by default | Kept out on purpose | Admin uses the status filter on the right |
| No bulk Reject with a reason | Kept out on purpose | Reject from the listing's edit page |
| Prices shown as whole Taka | `price_text` uses `%d` | Use a decimal format if paisa matter |
| Local only: `DEBUG=1`, SQLite, media served by Django | It is a local student project | See deployment steps below |

## 16.2 Steps to deploy for real

1. `DEBUG=0`, set `ALLOWED_HOSTS`, strong random `SECRET_KEY`.
2. Switch to PostgreSQL in `DATABASES`.
3. Run with Gunicorn behind Nginx, with HTTPS.
4. Serve `media/` from Nginx or cloud storage, run `collectstatic`.
5. Store `.env` values as server environment variables.
6. Add rate limiting and email password reset.

## 16.3 Feature ideas

- Wishlist / save listings
- Report a listing or user
- Seller profile page with average rating
- Filter by price range
- Notification badge for new messages and requests
- Image thumbnails to make the browse page faster
