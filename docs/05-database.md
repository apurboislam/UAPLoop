# 5. Database

The database is SQLite: one file, `db.sqlite3`. We never write SQL by hand. We write **models** (Python classes) and Django creates the tables.

## 5.1 All tables and how they connect

```
                 ┌──────────────┐
                 │   Category   │
                 └──────┬───────┘
                        │ 1 category has many listings
                        ▼
┌────────┐  seller  ┌──────────────┐  1 to many  ┌──────────────┐
│  User  │─────────▶│   Listing    │────────────▶│ ListingImage │
└───┬────┘  1 : N   └──────┬───────┘             └──────────────┘
    │                      │
    │ buyer         ┌──────┴────────┐
    ├──────────────▶│  BuyRequest   │ (one per listing + buyer)
    │               └──────┬────────┘
    │                      │ 1 to many
    │ sender        ┌──────▼────────┐
    ├──────────────▶│    Message    │
    │               └───────────────┘
    │ reviewer      ┌──────────────┐
    └──────────────▶│    Review    │◀── listing (one per listing + reviewer)
                    └──────────────┘
```

In words:

- A **User** can sell many **Listings**.
- A **Category** has many **Listings**. Each listing has exactly one category.
- A **Listing** has many **ListingImages** (photos).
- A **BuyRequest** links one buyer (User) to one Listing. A buyer can request a listing only once.
- A **Message** belongs to one BuyRequest. So each buy request is its own chat room.
- A **Review** links one reviewer (User) to one Listing, once.

## 5.2 User (`accounts/models.py`)

We replaced Django's default user with our own, because we log in with **email** (not a username) and need student fields.

| Field | Type | Notes |
|-------|------|-------|
| `id` | auto number | Primary key |
| `full_name` | text (150) | Student's name |
| `university_id` | text (20), unique | College / university ID |
| `phone` | text (20), unique | Only digits are stored (see below) |
| `status` | `active` / `inactive` / `banned` | Default `active` |
| `email` | email, unique | Used to log in |
| `registered_at` | date and time, empty allowed | Empty means the student has not registered yet |
| `password` | hashed text | From Django's `AbstractBaseUser`. Never stored as plain text. |
| `is_staff` | yes/no | Can enter the admin panel |
| `is_superuser` | yes/no | Has all permissions (from `PermissionsMixin`) |
| `is_active` | yes/no | Django's own "can log in" flag |
| `created_at` | date and time | Filled automatically |

`USERNAME_FIELD = 'email'` tells Django "the email is the login name".

The `save()` method does two small jobs every time a user is saved:

```python
def save(self, *args, **kwargs):
    if self.phone:
        self.phone = ''.join(c for c in self.phone if c.isdigit())
    self.is_active = self.status == 'active'
    super().save(*args, **kwargs)
```

1. **Cleans the phone**: `017-0000 0001` becomes `01700000001`, so matching during registration is reliable.
2. **Syncs `is_active` with `status`**: Django's login system already refuses users whose `is_active` is False. So by setting status to `inactive` or `banned`, the admin automatically blocks login. We did not write any extra login check.

Why are `university_id`, `phone` and `email` allowed to be empty (`null=True`)? Because the admin account is created from `.env` with only an email and password. Students always have all of them.

`UserManager.create_superuser` exists because Django's `createsuperuser` command expects it. We normally use our own `ensure_admin` command instead.

## 5.3 Category (`catalog/models.py`)

| Field | Notes |
|-------|-------|
| `name` | Unique, like "Electronics" |
| `slug` | URL-friendly version, like `electronics`. Used in the browse filter `?category=electronics`. |
| `is_active` | Admin can hide a category without deleting it |
| `sort_order` | Lower number shows first |

Ordering is `sort_order`, then `name`.

## 5.4 Listing (`catalog/models.py`)

| Field | Notes |
|-------|-------|
| `seller` | Link to User. `PROTECT`: a user with listings cannot be deleted (ban them instead) |
| `category` | Link to Category. `PROTECT`: a category with listings cannot be deleted (hide it instead) |
| `title` | Up to 120 characters |
| `description` | Long text |
| `listing_type` | `sell`, `exchange`, `rent`, `donate` |
| `price` | Decimal, empty for exchange and donate |
| `rent_period` | `day`, `week`, `month`, only for rent |
| `exchange_for` | What the seller wants in exchange |
| `condition` | `new`, `like_new`, `good`, `fair` |
| `status` | `pending` (default), `approved`, `rejected` |
| `reject_reason` | Admin writes why it was rejected; the seller sees it on "My listings" |
| `is_sold` | True after sold / exchanged / donated |
| `reviewed_by`, `reviewed_at` | Which admin approved or rejected it, and when |
| `created_at`, `updated_at` | Filled automatically |

### The moderation rule in one place

```python
class ApprovedQuerySet(models.QuerySet):
    def public(self):
        return self.filter(status='approved')

class Listing(models.Model):
    ...
    objects = ApprovedQuerySet.as_manager()
```

Every public page uses `Listing.objects.public()`. The rule "only approved listings are visible" is written **once**. If we wrote `filter(status='approved')` in every view, one forgotten view would leak unapproved listings. The same queryset is reused by `Review`.

### Database constraints (safety net)

```python
models.CheckConstraint(condition=Q(listing_type__in=['exchange', 'donate']) | Q(price__isnull=False), name='sell_rent_need_price')
models.CheckConstraint(condition=~Q(listing_type='rent') | Q(rent_period__isnull=False), name='rent_needs_period')
```

Read them as:

- "Either the type is exchange/donate, **or** the price is filled." So sell and rent must have a price.
- "Either it is **not** rent, **or** the rent period is filled." So rent must have a period.

The form already checks these, but the database checks them again. Even if a bug or the admin panel tried to save bad data, the database refuses.

### Indexes

`Index(fields=['status', '-created_at'])` and `Index(fields=['category', 'status'])` make the most common searches fast: "newest approved listings" and "approved listings in this category". An index is like the index at the back of a book.

### Helper methods (used in templates)

| Method | Returns |
|--------|---------|
| `cover()` | First photo, used on browse cards |
| `done_text()` | "Donated", "Exchanged" or "Sold" depending on type |
| `price_text()` | "Free", "Tk 500" or "Tk 100 / week" |

## 5.5 ListingImage

| Field | Notes |
|-------|-------|
| `listing` | Link to Listing. `CASCADE`: deleting the listing deletes these rows |
| `image` | The file, stored under `media/listings/<listing id>/<random name>.<ext>` |
| `position` | Upload order, first one is the cover |

`image_path()` gives every photo a **random name** (`uuid4`). Why? Two students may upload `IMG_001.jpg`; random names never clash, and the original file name (which may contain personal info) is not kept.

## 5.6 BuyRequest

| Field | Notes |
|-------|-------|
| `listing` | Which item |
| `buyer` | Who wants it |
| `amount` | Offered amount, only for sell and rent |
| `status` | `pending` or `accepted` |
| `created_at` | When |

`UniqueConstraint(fields=['listing', 'buyer'])`: one buyer can make only one request per listing.

A BuyRequest is used for three things: unlocking the seller's phone, being a chat room, and proving a buyer is "verified" for reviews.

## 5.7 Message (`inbox/models.py`)

| Field | Notes |
|-------|-------|
| `buy_request` | Which chat it belongs to |
| `sender` | Who wrote it |
| `text` | Up to 1000 characters |
| `created_at` | When; messages are ordered oldest first |

## 5.8 Review (`reviews/models.py`)

| Field | Notes |
|-------|-------|
| `listing` | Which item |
| `reviewer` | Who wrote it |
| `rating` | 1 to 5, displayed as stars like ★★★☆☆ |
| `comment` | Optional text |
| `status` | pending / approved / rejected (same choices as Listing) |

`UniqueConstraint(fields=['listing', 'reviewer'])`: one review per buyer per item.

The star labels are built with one line: `[(n, '★' * n + '☆' * (5 - n)) for n in range(1, 6)]`. For n=3 it gives `★★★☆☆`.

## 5.9 What happens when something is deleted

| Deleted | Effect | Reason |
|---------|--------|--------|
| Listing | Its images, buy requests, messages and reviews are deleted too (`CASCADE`) | They mean nothing without the listing |
| User with listings | **Blocked** (`PROTECT`) | We do not want to lose listings by accident. Ban the user instead. |
| Category with listings | **Blocked** (`PROTECT`) | Set `is_active` off instead |
| Admin who reviewed a listing | `reviewed_by` becomes empty (`SET_NULL`) | The listing should stay |

## 5.10 Migrations

There are 5 migration files across the apps. They were generated with `makemigrations`. Never edit them by hand; change the model and generate a new one.
