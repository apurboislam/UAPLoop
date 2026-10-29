# 9. Reviews

Folder: `reviews/`

## 9.1 Rules

1. Only a **verified buyer** can review: someone whose buy request for that listing was **accepted** by the seller.
2. One review per buyer per listing.
3. A new review is **pending** and hidden until the admin approves it.
4. The reviewer sees their own pending review status ("Your review is pending...").

Why so strict? Fake reviews are a big problem on marketplaces. A review here is proof that a real student actually got the item.

## 9.2 `review_context()` (shared helper)

```python
def review_context(request, listing):
    my_review = None
    can_review = False
    if request.user.is_authenticated:
        my_review = listing.reviews.filter(reviewer=request.user).first()
        verified = listing.requests.filter(buyer=request.user, status='accepted').exists()
        can_review = verified and my_review is None
    return {
        'reviews': listing.reviews.public().select_related('reviewer'),
        'my_review': my_review,
        'can_review': can_review,
        'review_form': ReviewForm(),
    }
```

It returns everything the review section needs:

- `reviews`: only **approved** reviews (`public()`, the same queryset used by listings).
- `can_review`: True only if verified and not reviewed yet.

The catalog `detail` view calls it and adds the result to its own context. `detail.html` then includes `reviews/section.html`. This keeps review logic inside the reviews app.

## 9.3 `add_review` view

```python
@login_required
def add_review(request, pk):
    listing = get_object_or_404(Listing.objects.public(), pk=pk)
    if request.method == 'POST' and review_context(request, listing)['can_review']:
        form = ReviewForm(request.POST)
        if form.is_valid():
            review = form.save(commit=False)
            review.listing = listing
            review.reviewer = request.user
            review.save()
            messages.success(request, 'Review submitted. It will appear after admin approval.')
    return redirect('detail', pk=pk)
```

The important part: it checks `can_review` **again on the server**. Hiding the form in HTML is not enough, because anyone could send a POST request by hand. **Never trust the browser.**

## 9.4 Stars

The rating choices are `1: ★☆☆☆☆` up to `5: ★★★★★`. The form shows them in a dropdown, and the page shows them in yellow (`text-warning`) using `get_rating_display`, a method Django creates automatically for any field with choices.

## 9.5 Admin

Reviews admin lists listing, reviewer, rating, status, date, with a status filter and an **Approve reviews** bulk action. To reject, open the review and set status to Rejected.
