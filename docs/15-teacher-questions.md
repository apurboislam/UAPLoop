# 15. Questions a Teacher May Ask (With Answers)

Grouped by topic. Short, direct answers you can say out loud, with the file to show if asked.

---

## A. About the project

**Q1. What is your project?**
UAPLoop is a marketplace website for University of Asia Pacific students to sell, exchange, rent or donate personal items. Only verified students can use it, and the admin approves every listing and review.

**Q2. What problem does it solve?**
Students trade things in messy Facebook groups with fake accounts and no trust. UAPLoop allows only real students, checks every post, protects phone numbers, and allows reviews only from real buyers.

**Q3. Who are the users?**
Visitors (browse only), students (post, request, chat, review) and the admin (manages students, categories, approvals).

**Q4. What technologies did you use and why?**
Python and Django for the backend, because Django gives login, admin panel, forms, ORM and security built in. SQLite as the database because it is a single file with no setup. Bootstrap 5 for a responsive design without writing CSS. Pillow to verify uploaded images.

**Q5. Is it a static or dynamic website?**
Dynamic. Pages are built on the server for each request using data from the database, and they change depending on who is logged in.

**Q6. What architecture does it follow?**
Django's MVT (Model, View, Template). Models define data, views contain the logic, templates show HTML. URLs connect addresses to views. It is a server-rendered, monolithic app split into 4 apps: accounts, catalog, inbox, reviews.

**Q7. What is the difference between MVT and MVC?**
They are almost the same idea. In MVC the Controller handles requests. In Django, the framework itself plus `urls.py` acts like the controller, Django's "View" does the controller's logic, and the "Template" is what MVC calls the View.

**Q8. Why did you split the project into 4 apps?**
Each app handles one concern, so the code is easier to find, test and explain. accounts = users, catalog = listings and requests, inbox = chat, reviews = reviews.

---

## B. Database

**Q9. Which database and why?**
SQLite. It is built into Python, stored in one file (`db.sqlite3`), needs no server, and is enough for a local university project. Because we use Django's ORM, we could switch to PostgreSQL by changing only `DATABASES` in settings.

**Q10. How many tables / models are there?**
Seven of our own: User, Category, Listing, ListingImage, BuyRequest, Message, Review. Django adds its own tables for sessions, permissions and admin logs.

**Q11. Explain the relationships.**
User 1 to many Listing (seller). Category 1 to many Listing. Listing 1 to many ListingImage. Listing and User connect through BuyRequest (many to many with extra data like amount and status). BuyRequest 1 to many Message. Listing and User connect through Review. See [05-database.md](05-database.md).

**Q12. What is a ForeignKey?**
A column that stores the id of a row in another table, creating a link. For example `Listing.seller` stores the id of a User.

**Q13. What do CASCADE, PROTECT and SET_NULL mean?**
They decide what happens when the linked row is deleted. CASCADE deletes the dependent rows too (deleting a listing deletes its photos). PROTECT blocks the delete (you cannot delete a user who has listings). SET_NULL empties the link (if the reviewing admin is deleted, the listing stays).

**Q14. What is the ORM? Why not write SQL?**
Object Relational Mapper: we write Python like `Listing.objects.filter(status='approved')` and Django generates SQL. It is shorter, works with any database, and automatically protects against SQL injection.

**Q15. What is a migration?**
A file Django creates from our models that describes how to build or change the tables. `makemigrations` creates it, `migrate` applies it to the database.

**Q16. What are constraints and why use them when the form already validates?**
Rules enforced by the database itself: one request per buyer per listing, one review per buyer per listing, sell/rent must have a price, rent must have a period. The form gives friendly messages; the database is the final safety net if any code path (like the admin panel or a bug) tries to save bad data.

**Q17. What are the indexes for?**
They speed up the most common queries: approved listings sorted by newest, and approved listings in a category. Like a book's index, the database finds rows without scanning everything.

**Q18. Is the data normalised?**
Yes. Each fact is stored once: categories in their own table, photos in their own table, users referenced by id. The only repeated value is a status string, which is normal.

**Q19. What is the N+1 problem and did you handle it?**
When showing 9 listings, loading each one's category and photo separately would cost 1 + 9 + 9 queries. We used `select_related` and `prefetch_related` in browse and other views, so Django loads them in a few queries.

---

## C. Authentication and users

**Q20. How does registration work?**
There is no open signup. The admin creates the student record. The student enters university ID, phone, name and email; if all match an active, unregistered record, they set a password. See `accounts/forms.py`.

**Q21. Why not let students sign up freely?**
The app must be UAP students only. The university already knows its students, so the admin creates the records and nobody else can join.

**Q22. Why do students log in with email if they register with university ID?**
University ID and phone prove identity once at registration. Email is easy to remember and unique, so it is the login name (`USERNAME_FIELD = 'email'`).

**Q23. Why a custom User model?**
Django's default user logs in with a username and has no phone, university ID or status. We needed email login and student fields. Django recommends deciding the custom user model before the first migration, so we did.

**Q24. How are passwords stored?**
Hashed with Django's default PBKDF2 algorithm plus a random salt. The real password is never saved, so even with the database file nobody can read it.

**Q25. How do you block a student?**
Set status to inactive or banned. `User.save()` sets `is_active = (status == 'active')`, and Django's login automatically rejects inactive users. Registration also checks status.

**Q26. How does the app remember a logged-in user?**
With a session. After login, Django stores the user id in a server-side session and gives the browser a cookie with the session id. Each request sends the cookie back.

**Q27. What does `@login_required` do?**
It checks if the user is logged in before running the view. If not, it redirects to `LOGIN_URL`, which is our 401 page, with `?next=` set to the requested page.

**Q28. What if a student forgets the password?**
The admin uses the "Reset registration" action. The student registers again with their details and sets a new password. (An email-based reset is a possible future feature.)

**Q29. How is the admin account created?**
With our custom command `manage.py ensure_admin`, which reads email and password from `.env` and creates or updates the superuser. Running it again is safe.

---

## D. Listings and features

**Q30. How does moderation work?**
New listings are saved with status `pending`. Public pages only use `Listing.objects.public()`, which filters approved ones. The admin approves in bulk or rejects with a reason that the seller sees.

**Q31. Why put the "approved only" filter in a custom QuerySet?**
So the rule exists in one place. If every view wrote its own filter, forgetting it once would leak hidden listings. Reviews reuse the same QuerySet.

**Q32. Can a user see someone else's pending listing by changing the URL?**
No. `detail` uses `get_object_or_404(Listing.objects.public(), pk=pk)`, so a non-approved listing gives 404.

**Q33. How does search work?**
`title__icontains=search`: a case-insensitive "contains" match on the title. Combined with optional category and type filters, all from the URL.

**Q34. How does pagination work?**
Django's `Paginator` splits results into pages of 9. The template shows Previous/Next links and keeps the current filters in the URL.

**Q35. How are photos uploaded and stored?**
The form uses `multipart/form-data`. We check at most 5 files, at most 5 MB each, and that Pillow can open them as images. Each is saved under `media/listings/<listing id>/` with a random UUID file name.

**Q36. Why random file names?**
To avoid clashes when two people upload `IMG_001.jpg`, to avoid unsafe names, and to not leak the original name.

**Q37. What happens to photos when a listing is removed?**
The view deletes each file from disk before deleting the listing, because Django deletes database rows but not files.

**Q38. Why can't sellers edit listings?**
To keep it simple and safe: an edited listing would need re-approval, or a seller could change an approved listing into something bad. Sellers can remove and post again.

**Q39. What is the difference between "Accept request" and "Mark sold"?**
Accept is for a buyer inside the app: it marks the request accepted (which enables the buyer's review) and marks the item sold. Mark sold is for items sold outside the app.

**Q40. How do the four listing types differ in the code?**
The form's `clean()` method requires a price for sell and rent, a period for rent, and "exchange for" for exchange. It clears unused fields. `price_text()` shows "Free" for donate, and `done_text()` shows Sold, Exchanged or Donated.

---

## E. Buy requests, chat and reviews

**Q41. Why is the phone number hidden at first?**
Privacy. Only students who show real interest by sending a request see the seller's phone, and only the seller sees the buyer's phone.

**Q42. How does the chat work?**
Each BuyRequest is a chat room. Messages are rows linked to it. The page sends a request to `/inbox/<id>/new/?after=<last id>` every 0.5 seconds and adds any new messages with JavaScript. This is called polling.

**Q43. Why polling and not WebSockets?**
WebSockets need Django Channels, Redis and a different server. Polling needs one small view and about 40 lines of plain JavaScript and works with the normal server. The half-second delay is fine for a student marketplace.

**Q44. Doesn't polling every 0.5 s load the server?**
A little, but each call is tiny (only messages newer than the last id), it pauses when the browser tab is hidden, and it waits for each answer before asking again. For a local university app this is acceptable. For many users we would increase the interval or move to WebSockets.

**Q45. Can a third person read a chat?**
No. All inbox views get the BuyRequest through `my_conversations(user)`, which only includes requests where the user is the buyer or the seller. Others get 404. A test checks this.

**Q46. Who can write a review?**
Only a buyer whose request for that listing was accepted, once per listing. The server checks this again on submit, and the review is hidden until the admin approves it.

**Q47. Why check `can_review` in the view if the form is already hidden?**
Because anyone can send a POST request without using our page. Security checks must happen on the server; the HTML only decides what to show.

---

## F. Security

**Q48. What security measures did you take?**
Admin-provisioned accounts, hashed passwords, CSRF tokens, POST-only actions, ownership checks in every owner view, approved-only public queries, auto-escaped templates, safe JavaScript (`textContent`), ORM against SQL injection, open-redirect protection on login, image verification and size limits, random file names, and secrets in `.env`. Full table in [12-security.md](12-security.md).

**Q49. What is CSRF and how is it prevented?**
Cross-Site Request Forgery: another website tricks your browser into submitting a form to our site while you are logged in. Django adds a secret token to each form (`{% csrf_token %}`) and rejects POSTs without the right token.

**Q50. What is XSS and how is it prevented?**
Cross-Site Scripting: a user types `<script>` into a title or message so it runs in other people's browsers. Django templates escape all values automatically, and our chat JavaScript inserts text with `textContent`, never `innerHTML`.

**Q51. What is SQL injection and how is it prevented?**
Typing SQL code into an input to change the query. The Django ORM always sends user values as parameters, separate from the SQL text, so they are never executed as code.

**Q52. How do you stop a student from deleting another student's listing?**
Every owner action fetches with `get_object_or_404(Listing, pk=pk, seller=request.user)`. For someone else's listing, the query finds nothing and returns 404.

**Q53. Why is logout a POST request?**
So other websites cannot log users out with a simple link or image tag. Django also recommends POST for logout.

**Q54. What is an open redirect and where did you handle it?**
When a site redirects to any URL from a parameter, attackers can send users to fake sites. On login we only follow `next` if `url_has_allowed_host_and_scheme` says it is our own site.

**Q55. Where are secrets stored?**
In `.env` (secret key, admin email and password), read by a small `read_env()` function in settings. `.env` is in `.gitignore` so it is never committed.

---

## G. Frontend

**Q56. Is it responsive? How?**
Yes. Bootstrap's grid classes change the number of columns by screen size (2 cards per row on phones, up to 6 on desktop), the navbar collapses on phones, tables scroll horizontally, and the viewport meta tag is set.

**Q57. Why no custom CSS?**
Project rule for simplicity. Bootstrap covers everything we need and gives a consistent look.

**Q58. How do all pages share the same navbar?**
Template inheritance: every page `extends 'base.html'` and only fills the `content` block.

**Q59. How do success messages appear?**
Views call `messages.success(request, '...')`. Django's messages framework stores it in the session, and `base.html` shows it as a Bootstrap alert on the next page.

**Q60. Is there any JavaScript?**
Only Bootstrap's bundle (for the collapsing navbar) and the small polling script on the chat page.

---

## H. Testing and quality

**Q61. How did you test the project?**
16 automated Django tests covering registration, error pages, login redirects, moderation, listing rules, contact privacy, chat privacy, the live-chat endpoint and review rules. All pass. Plus manual testing in the browser.

**Q62. Do tests touch the real database?**
No. Django creates a temporary test database and deletes it afterwards.

**Q63. Why no comments in the code?**
Project rule: code should be simple and clear enough to read without comments, with descriptive names. The explanation lives in these docs instead.

**Q64. Why function-based views instead of class-based views?**
They are plain Python functions, easy to read top to bottom, and good for beginners. Class-based views hide logic in parent classes.

---

## I. Design decisions and future

**Q65. What was the hardest part?**
Possible answers: making registration match an existing record securely; making the live chat work without extra libraries; enforcing "approved only" safely everywhere.

**Q66. How would you deploy it online?**
Set `DEBUG=0`, set `ALLOWED_HOSTS`, use a strong `SECRET_KEY`, use PostgreSQL, run with Gunicorn behind Nginx with HTTPS, serve media files from Nginx or cloud storage, and run `collectstatic`.

**Q67. Can it handle many users?**
For a single university, yes, after moving to PostgreSQL. For much larger scale we would add caching, move chat to WebSockets and photos to cloud storage.

**Q68. What would you add next?**
Login/registration rate limiting, password reset by email, editing listings with re-approval, notifications for new messages, multiple categories filter, and a report-listing button. See [16-limitations-and-future.md](16-limitations-and-future.md).

**Q69. What did you learn?**
Django's MVT structure, ORM and migrations, custom user models, form validation, authentication and sessions, web security basics (CSRF, XSS, SQL injection), file uploads, AJAX polling with fetch, responsive design with Bootstrap, and automated testing.

**Q70. Show me where X happens. (Quick lookup)**

| Feature | File and function |
|---------|-------------------|
| Registration matching | `accounts/forms.py`, `RegisterForm.clean` |
| Ban blocks login | `accounts/models.py`, `User.save` |
| Admin from `.env` | `accounts/management/commands/ensure_admin.py` |
| Approved-only rule | `catalog/models.py`, `ApprovedQuerySet.public` |
| Listing validation | `catalog/forms.py`, `ListingForm.clean` |
| Search and filters | `catalog/views.py`, `browse` |
| Contact reveal | `catalog/templates/catalog/detail.html`, `my_request` block |
| Accept request | `catalog/views.py`, `accept_request` |
| Chat privacy | `inbox/views.py`, `my_conversations` |
| Live chat | `inbox/views.py`, `new_messages` + script in `conversation.html` |
| Review permission | `reviews/views.py`, `review_context` |
| Approve actions | `catalog/admin.py`, `reviews/admin.py` |
| Bootstrap on forms | `config/forms.py`, `add_bootstrap` |
| `.env` reading | `config/settings.py`, `read_env` |
