# 8. Buy Requests and Inbox (Chat)

## 8.1 Why buy requests exist

We did not want a seller's phone number visible to everyone. Instead:

1. A buyer clicks **Request to buy** (and enters an amount for sell or rent).
2. This creates a `BuyRequest`.
3. Now the buyer sees the seller's name and phone, and a private chat opens.
4. The seller sees the buyer's name, phone and offer on the "Buy requests" page.
5. The seller clicks **Accept** on the best request. The item becomes Sold and that buyer can write a review.

One `BuyRequest` = one contact unlock + one chat room + one proof of purchase.

## 8.2 Sending a request (`request_buy` in `catalog/views.py`)

```python
@login_required
def request_buy(request, pk):
    listing = get_object_or_404(Listing.objects.public(), pk=pk)
    if request.method == 'POST' and not listing.is_sold and listing.seller != request.user:
        form = BuyRequestForm(request.POST, listing=listing)
        if form.is_valid():
            BuyRequest.objects.get_or_create(listing=listing, buyer=request.user, defaults={'amount': form.cleaned_data['amount']})
            messages.success(request, 'Request sent. The seller contact is shown below.')
        else:
            messages.error(request, 'Enter a valid amount.')
    return redirect('detail', pk=pk)
```

Checks: the listing must be approved, not sold, and **not your own**. `get_or_create` means sending twice does not create a duplicate (the database's unique constraint also forbids it).

`BuyRequestForm` requires an amount for sell and rent, and clears it for exchange and donate.

## 8.3 Seeing and accepting requests

- `listing_requests`: shows all requests for **your** listing (`seller=request.user` check), with buyer name, phone, amount, status, a Chat link and an Accept button.
- `accept_request`: POST only, and the request must belong to your listing (`listing__seller=request.user`). It sets the request to `accepted` and the listing to `is_sold = True`.

The Accept button only shows while the request is pending and the listing is not sold. The view also checks `is_sold` itself, so once one request is accepted, no other request can be accepted, even by sending a POST by hand.

## 8.4 The inbox (`inbox/views.py`)

### Who can see a conversation?

```python
def my_conversations(user):
    mine = Q(buyer=user) | Q(listing__seller=user)
    return BuyRequest.objects.filter(mine).select_related('listing', 'buyer', 'listing__seller')
```

"All buy requests where I am the buyer **or** I am the seller." `Q` objects let us write OR in a query.

All three inbox views start from this function, so only the two people in a chat can ever open it. Anyone else gets 404. This is tested.

### Inbox page

```python
conversations = my_conversations(request.user).annotate(last=Max('messages__created_at')).order_by('-last', '-created_at')
```

`annotate(last=Max(...))` adds the time of the newest message to each conversation, so the most recently active chats appear first, like WhatsApp.

### Conversation page

- Shows the item title, the other person's name and phone, all messages (yours on the right).
- A form to send a new message. After sending, it redirects to the same page (Post/Redirect/Get).
- `last_id` (the id of the newest message) is passed to the page for the live update.

## 8.5 Live chat: how new messages appear without refreshing

We did **not** use WebSockets or any extra library. We used **polling**: the page asks the server every half second "anything new?".

### Server side: `new_messages` view

URL: `/inbox/<id>/new/?after=<last message id>`

```python
after = request.GET.get('after', '')
after = int(after) if after.isdigit() else 0
for message in buy_request.messages.filter(id__gt=after).select_related('sender'):
    found.append({'id': ..., 'name': ..., 'mine': ..., 'time': ..., 'text': ...})
return JsonResponse({'messages': found})
```

It returns only messages with an id **greater than** `after`, as **JSON** (a text format JavaScript can read). If `after` is not a number, it safely uses 0.

### Browser side: JavaScript in `conversation.html`

```
poll()
  ├─ if the tab is hidden: wait 0.5 s, try again (saves server work)
  ├─ fetch /inbox/<id>/new/?after=<after>
  ├─ for each new message: add it to the page, set after = its id
  └─ wait 0.5 s, call poll() again
```

Important details:

- `after` starts at `last_id` from the server, so messages already on the page are not added twice.
- New messages are built with `document.createElement` and `textContent`, **never** `innerHTML`. So if someone sends `<script>alert(1)</script>` it shows as plain text and does not run. This prevents an **XSS** attack.
- `.catch(function () {})`: if the network fails once, the loop just continues.
- `setTimeout` (not `setInterval`) waits until the previous request has finished, so requests never pile up.

### Why polling instead of WebSockets?

| | Polling (what we use) | WebSockets |
|---|---|---|
| Extra packages | None | Django Channels, Redis |
| Server setup | Normal `runserver` | Special ASGI server |
| Code size | About 40 lines of JS + 1 view | Much more |
| Speed | Up to 0.5 s delay | Instant |
| Fits a small local project? | Yes | Overkill |
