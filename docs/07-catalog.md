# 7. Catalog: Listings

Folder: `catalog/`. This is the heart of the app.

## 7.1 Life of a listing

```
Student submits ──▶ PENDING ──admin approves──▶ APPROVED ──seller accepts request──▶ APPROVED + SOLD
                       │                           │        or "Mark sold"
                       └──admin rejects──▶ REJECTED (with reason)
At any time the seller can REMOVE (delete) it.
```

- **Pending** and **rejected** listings are visible only to their seller (on "My listings") and the admin.
- **Approved** listings are visible to everyone.
- **Sold** listings stay visible with a red "Sold" / "Exchanged" / "Donated" badge, but nobody can request them.

## 7.2 Creating a listing (`new_listing` view, `ListingForm`)

```python
@login_required
def new_listing(request):
    form = ListingForm(request.POST or None, request.FILES or None)
    if form.is_valid():
        listing = form.save(commit=False)
        listing.seller = request.user
        listing.save()
        for position, photo in enumerate(request.FILES.getlist('photos')):
            ListingImage.objects.create(listing=listing, image=photo, position=position)
        messages.success(request, 'Listing submitted. It will appear after admin approval.')
        return redirect('my_listings')
    return render(request, 'catalog/new.html', {'form': form})
```

Explained:

1. `@login_required`: only logged-in students.
2. `request.POST or None`: on first visit (GET) the form is empty; on submit (POST) it gets the data. This one line handles both cases.
3. `form.save(commit=False)`: build the listing but do not save yet, because we must add the seller first.
4. `listing.seller = request.user`: the seller is **always** the logged-in user. It is not a form field, so nobody can post as someone else.
5. Status is not set, so it uses the default `pending`.
6. Each uploaded photo becomes a `ListingImage` with its position.
7. Show a message and go to "My listings". This is the **Post/Redirect/Get** pattern: after a POST we redirect, so pressing refresh does not submit the form twice.

### The form's checks (`ListingForm.clean`)

| Rule | Why |
|------|-----|
| Category list shows only active categories | Admin can hide a category |
| Sell and rent need a price | You must know the cost |
| Rent needs a rent period | "Tk 100" means nothing without "per day/week/month" |
| Exchange needs "exchange for" | Buyers must know what the seller wants |
| Non-rent: rent period is cleared | Avoid leftover junk data |
| Exchange and donate: price is cleared | They have no price |
| At most 5 photos | Saves disk space |
| Each photo at most 5 MB | Saves disk space |
| Each photo must be a real image | `forms.ImageField().clean(photo)` uses Pillow to open it. A `.exe` renamed to `.jpg` is rejected. |

The photo input is written by hand in `new.html` (`<input type="file" name="photos" multiple>`) because one file field that accepts several files is simplest this way. The form tag has `enctype="multipart/form-data"`, which is required for file uploads.

## 7.3 Browsing (`browse` view)

```python
listings = Listing.objects.public().select_related('category').prefetch_related('images')
```

- `public()`: only approved listings.
- `select_related('category')` and `prefetch_related('images')`: load categories and photos together in a few queries instead of one query per card. Without them, 9 cards would cause about 18 extra database queries (the "N+1 problem").

Then optional filters from the URL:

| URL part | Filter |
|----------|--------|
| `?category=electronics` | `category__slug=...` |
| `?type=rent` | `listing_type=...` |
| `?q=casio` | `title__icontains=...` (title contains the text, ignoring capitals) |

**Pagination**: `Paginator(listings, 9)` shows 9 listings per page. `get_page()` safely handles bad page numbers (like `?page=abc` or `?page=999`).

To keep filters when moving to the next page, the view copies the current URL query, removes `page`, and gives it to the template:

```python
query = request.GET.copy()
query.pop('page', None)
```

The Next link becomes `?q=casio&type=rent&page=2`.

## 7.4 Detail page (`detail` view)

```python
listing = get_object_or_404(Listing.objects.public(), pk=pk)
```

If the listing does not exist **or is not approved**, the visitor gets a 404. A pending listing cannot be seen even if someone guesses its number.

The page shows a different box depending on who is looking:

| Who | What they see |
|-----|---------------|
| The seller | Link "See buy requests" |
| A student who already requested | Seller name and **phone**, their offer and status, link to the chat |
| Anyone, if the item is sold | "This item has been sold." |
| Another logged-in student | "Request to buy" form (with amount for sell/rent) |
| Visitor | "Login to request this item." |

Reviews are added to the page through `review_context()` from the reviews app (see [09-reviews.md](09-reviews.md)).

## 7.5 My listings (`my_listings` view)

Shows **all** of the student's own listings (any status) in a table: title, status badge (green approved, red rejected, yellow pending), reject reason, "Mark sold" button, number of requests, Remove link.

Note it uses `Listing.objects.filter(seller=request.user)`, **not** `public()`, because sellers must see their own pending and rejected listings.

## 7.6 Mark sold (`mark_sold` view)

For when the item was sold outside the app. POST only, and only for your own listing:

```python
listing = get_object_or_404(Listing, pk=pk, seller=request.user)
```

The `seller=request.user` part is the **ownership check**. If student A tries `/sold/7/` on student B's listing, the query finds nothing and returns 404.

## 7.7 Remove (`remove_listing` view)

- GET shows a confirmation page: "Are you sure? This cannot be undone."
- POST deletes the photo **files** from disk first, then deletes the listing. The database then deletes its images, buy requests, messages and reviews (CASCADE).

Why delete files by hand? Django deletes database rows but **not** the files on disk. Without this loop, `media/` would fill with orphan photos.

## 7.8 Admin for listings (`catalog/admin.py`)

- Categories: the slug is filled automatically while typing the name (`prepopulated_fields`).
- Listings: list with filters by status, sold, type and category; search by title; photos shown inline.
- **Approve** action: select many pending listings and approve them in one click. It also records which admin approved and when.
- To **reject**: open a listing, change status to Rejected and type a reason. `save_model` records `reviewed_by` and `reviewed_at` whenever the status changes.
- Buy requests are also registered so the admin can view them.
