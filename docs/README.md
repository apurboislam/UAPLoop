# UAPLoop Documentation

Welcome. This folder explains the whole UAPLoop project in simple words. You only need to know the basics of programming (variables, functions, if/else, loops) to follow it.

## Read in this order

| # | File | What you learn |
|---|------|----------------|
| 1 | [01-overview.md](01-overview.md) | What UAPLoop is, who it is for, what it can do |
| 2 | [02-web-and-django-basics.md](02-web-and-django-basics.md) | How websites work and how Django works (beginner crash course) |
| 3 | [03-setup-and-run.md](03-setup-and-run.md) | How to install and run the project on your computer |
| 4 | [04-project-structure.md](04-project-structure.md) | Every folder and file, and why it exists |
| 5 | [05-database.md](05-database.md) | All tables (models), their columns and how they connect |
| 6 | [06-accounts.md](06-accounts.md) | Users, registration, login, logout, the admin account |
| 7 | [07-catalog.md](07-catalog.md) | Listings: create, browse, search, approve, sell, remove |
| 8 | [08-buy-requests-and-inbox.md](08-buy-requests-and-inbox.md) | Buy requests, contact sharing and the live chat |
| 9 | [09-reviews.md](09-reviews.md) | Verified-buyer reviews with admin approval |
| 10 | [10-admin-panel.md](10-admin-panel.md) | What the admin does and how |
| 11 | [11-templates-and-ui.md](11-templates-and-ui.md) | HTML pages, Bootstrap, responsive design |
| 12 | [12-security.md](12-security.md) | How the app protects users and data |
| 13 | [13-testing.md](13-testing.md) | Automated tests and what they prove |
| 14 | [14-full-walkthrough.md](14-full-walkthrough.md) | One complete story from admin setup to review, step by step |
| 15 | [15-teacher-questions.md](15-teacher-questions.md) | Questions a teacher may ask, with answers |
| 16 | [16-limitations-and-future.md](16-limitations-and-future.md) | What is missing and how to improve it |

## The project in one paragraph

UAPLoop is a website where students of University of Asia Pacific (UAP) can **sell, exchange, rent or donate** their personal things (calculators, phones, books, rackets, anything). Only real students can join, because the admin creates each student record first. Every new listing is hidden until the admin approves it. A buyer sends a "buy request", which unlocks the seller's phone number and opens a private chat. When the seller accepts the request, the item is marked sold and the buyer may write a review.

## Tech stack at a glance

| Part | Tool | Why |
|------|------|-----|
| Language | Python 3 | Easy to read, popular in universities |
| Web framework | Django 6.1 | Comes with login, admin panel, database tools and forms built in |
| Database | SQLite | A single file (`db.sqlite3`), no server to install |
| Image handling | Pillow | Django needs it to check uploaded photos are real images |
| Design | Bootstrap 5 (from a CDN) | Ready-made responsive styles, no custom CSS needed |
| Photos storage | Local folder `media/` | Simple, no cloud account needed |

The older design notes written while planning the project live in `system-design/docs/`. This `docs/` folder is the beginner-friendly explanation of the finished code.
