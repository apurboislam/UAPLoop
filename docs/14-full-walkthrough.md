# 14. Full Walkthrough: One Story From Start to End

Follow Rahim (seller) and Karim (buyer) through the whole system. For each step: what the user does, which code runs, what changes in the database.

## Step 1: Admin prepares

| Who | Does | Code | Database |
|-----|------|------|----------|
| Admin | Runs `manage.py ensure_admin` | `ensure_admin.py` | User row for admin, `is_staff=True` |
| Admin | Adds category "Electronics" | Django admin, `CategoryAdmin` | Category row |
| Admin | Adds Rahim and Karim (name, ID, phone, email, Active) | `UserAdmin.save_model` | 2 User rows, unusable password, `registered_at` empty |

## Step 2: Rahim registers

| Does | Code | Result |
|------|------|--------|
| Opens `/register/`, types his details and a password | `register_view`, `RegisterForm.clean` | Details match the record |
| Submits | `RegisterForm.save` | Password hashed, `registered_at` = now |
| | `login(request, user)` | Session cookie created, Rahim is logged in |

## Step 3: Rahim posts a calculator

| Does | Code | Result |
|------|------|--------|
| Clicks New listing, fills "Casio fx-991", Sell, preferred handover place "UAP campus gate", Tk 800, 2 photos | `new_listing`, `ListingForm.clean` | Handover place and price present, photos valid |
| Clicks Submit listing | `form.save(commit=False)`, `seller = request.user` | Listing row with handover place, `status='pending'` |
| | loop creating `ListingImage` | 2 rows, files in `media/listings/<id>/` |
| Sees "My listings" | `my_listings` | Yellow "Pending" badge |

Karim opens the home page now: the calculator is **not** there, because `browse` uses `.public()`.

## Step 4: Admin approves

| Does | Code | Result |
|------|------|--------|
| Listings > select > Approve | `approve` action in `catalog/admin.py` | `status='approved'`, `reviewed_by`, `reviewed_at` |

## Step 5: Karim finds and requests it

| Does | Code | Result |
|------|------|--------|
| Registers like Rahim did | same as step 2 | |
| Searches "casio" on the Home page | `browse`, `title__icontains` | Card shows the handover place and **View item** button |
| Opens the listing | `detail` | Sees the handover place and **Send request** form |
| Offers Tk 750 and clicks Send request | `request_buy`, `BuyRequestForm` | BuyRequest row: Karim, Tk 750, pending |
| Page reloads | `detail` with `my_request` | Now shows Rahim's name and **phone**, plus **Chat with seller** |

## Step 6: They chat

| Does | Code | Result |
|------|------|--------|
| Karim opens the chat, writes "Is it still available?" | `conversation`, `MessageForm` | Message row |
| Rahim has the same chat open | JavaScript `poll()` calls `new_messages?after=0` every 0.5 s | Karim's message appears without refresh |
| Rahim replies | same | Karim sees it within half a second |
| Both check Inbox | `inbox`, `my_conversations` | One conversation listed |

## Step 7: Rahim accepts

| Does | Code | Result |
|------|------|--------|
| My listings > View requests (1) > Accept | `listing_requests`, `accept_request` | BuyRequest `accepted`, Listing `is_sold=True` |
| Home page | `browse.html` | Card shows red "Sold" badge |
| Other students open the listing | `detail.html` | "This item has been sold." No request form. |

## Step 8: Karim reviews

| Does | Code | Result |
|------|------|--------|
| Opens the listing | `review_context` | `can_review=True` (accepted request, no review yet) |
| Picks ★★★★★ and writes "Works perfectly" | `add_review` | Review row, `pending` |
| | `section.html` | Karim sees "Your review is pending" |
| Admin: Reviews > Approve reviews | `approve_reviews` | `approved` |
| Everyone opening the listing | `review_context` with `.public()` | Sees the 5-star review |

## Step 9 (optional): Problems

- If Rahim later misbehaves, admin sets his status to **Banned**. `User.save` sets `is_active=False`; he can no longer log in. His listings stay in the database (PROTECT).
- If Rahim removes a listing, the photos are deleted from disk and the listing with its requests, messages and reviews is deleted from the database.
