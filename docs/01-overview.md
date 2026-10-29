# 1. Overview: What is UAPLoop?

## The problem

University students often own things they no longer need: an old scientific calculator, last semester's books, a cricket bat. Other students need exactly those things. Facebook groups are messy: anyone can post, fake accounts exist, posts get lost, and there is no way to know if a seller is trustworthy.

## The solution

UAPLoop is a small marketplace **only for UAP students**. It gives:

1. **Trust**: only students the admin added can join. No fake accounts.
2. **Quality**: every listing is checked by the admin before anyone can see it.
3. **Privacy**: a seller's phone number is shown only to students who sent a buy request.
4. **Communication**: each buy request has its own private chat.
5. **Honest reviews**: only buyers whose request was accepted can write a review, and the admin approves it first.

## Who uses it (the "actors")

| Actor | What they can do |
|-------|------------------|
| **Visitor** (not logged in) | Browse approved listings, search, filter, view details and reviews |
| **Student** (logged in) | Everything a visitor can, plus: create listings, send buy requests, chat, accept requests on own listings, mark own items sold, remove own listings, write reviews |
| **Admin** | Uses the Django admin panel at `/admin/`: create student records, ban students, manage categories, approve or reject listings, approve reviews |

## The four listing types

| Type | Meaning | Price needed? | Extra field |
|------|---------|---------------|-------------|
| **Sell** | Give the item for money | Yes | none |
| **Exchange** | Swap for another item | No | "Wants in exchange" (required) |
| **Rent** | Lend for money for a time | Yes | Rent period: day, week or month (required) |
| **Donate** | Give it away free | No (shown as "Free") | none |

## Main features list

- Student "registration" that **claims** an admin-created record (no open signup)
- Login with email and password
- Create a listing with up to 5 photos (5 MB each)
- Admin moderation queue (pending, approved, rejected with a reason)
- Browse page with search by title, filter by category and type, pagination (9 per page)
- Listing detail page with photos, condition, category, description
- Buy request (with an offered amount for sell and rent)
- Phone number revealed after sending a request
- Private chat per buy request that updates live (every half second)
- Inbox listing all conversations
- Seller accepts a request, item becomes Sold / Exchanged / Donated
- Star reviews (1 to 5) from verified buyers, approved by admin
- Custom 401 (login required) and 404 (not found) pages
- Works on phones, tablets and computers (responsive Bootstrap)

## Why these choices (short answer)

- **Django** because it gives a login system, admin panel, forms and database layer for free. We wrote far less code.
- **SQLite** because the app runs locally for a university project. It is one file and needs no setup.
- **Function-based views and simple code** because the project is meant to be easy to read and explain.
- **Admin-created users** because the app must be UAP students only. The university already knows who its students are.

The rest of the docs explain each part in detail.
