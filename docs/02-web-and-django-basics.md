# 2. Web and Django Basics (Crash Course)

Read this if you have never built a website. It explains just enough to understand the rest.

## 2.1 How a website works

1. You type an address (a **URL**) like `http://127.0.0.1:8000/listing/5/` in the browser.
2. The browser sends a **request** to a **server** (a program running on some computer).
3. The server runs code, maybe reads a **database**, and builds a **response** (usually an HTML page).
4. The browser shows the HTML.

```
Browser  --- request (GET /listing/5/) --->  Server (Django)  ---> Database
Browser  <-- response (HTML page) --------  Server (Django)  <--- Database
```

### GET and POST

A request has a **method**:

- **GET**: "give me something". Opening a page, searching. Should never change data.
- **POST**: "here is some data, do something". Submitting a form, sending a message, logging out.

UAPLoop follows this rule: every action that changes data (create, accept, remove, logout, mark sold) only happens on POST.

### Status codes

Every response has a number:

| Code | Meaning | Where in UAPLoop |
|------|---------|------------------|
| 200 | OK | Normal pages |
| 302 | Redirect, "go to this other page" | After a form is saved |
| 401 | Not logged in | Our custom "Login required" page |
| 404 | Not found | Missing page, or someone else's private listing |

## 2.2 What is Django?

Django is a Python **framework**: a big toolbox that already solves common website problems (URLs, database, forms, login, admin panel, security). You plug your own code into it.

Django uses a pattern called **MVT**:

| Letter | Name | In plain words | In UAPLoop |
|--------|------|----------------|------------|
| M | **Model** | Describes a database table as a Python class | `Listing`, `User`, `Message`... in `models.py` |
| V | **View** | A Python function that receives a request and returns a response | `browse`, `detail`, `new_listing`... in `views.py` |
| T | **Template** | An HTML file with holes `{{ }}` that Django fills with data | `browse.html`, `detail.html`... |

Plus two helpers:

- **URLs** (`urls.py`): a list that says "this address runs this view".
- **Forms** (`forms.py`): classes that build HTML form fields and **check** (validate) what the user typed.

## 2.3 The life of one request in UAPLoop

Example: a visitor opens `/listing/5/`.

1. Django looks at `config/urls.py`, which includes `catalog/urls.py`.
2. It finds `path('listing/<int:pk>/', views.detail, name='detail')`. The `5` becomes `pk=5`.
3. It calls the function `detail(request, pk=5)` in `catalog/views.py`.
4. The view asks the database: "give me the approved listing with id 5" (`Listing.objects.public()`). If none exists, it shows the 404 page.
5. The view builds a **context** (a Python dictionary of data) and calls `render(request, 'catalog/detail.html', context)`.
6. The template fills the HTML with the listing title, photos, price, reviews.
7. Django sends the HTML back to the browser.

## 2.4 Important Django words

| Word | Meaning |
|------|---------|
| **Project** | The whole website. Ours has a settings folder called `config`. |
| **App** | One part of the project with its own models, views, templates. We have 4: `accounts`, `catalog`, `inbox`, `reviews`. |
| **ORM** | "Object Relational Mapper". Lets us write `Listing.objects.filter(status='approved')` in Python instead of SQL. Django turns it into SQL for us. |
| **QuerySet** | The result of an ORM query: a list-like group of rows. |
| **Migration** | A file Django generates that tells the database how to create or change tables to match our models. |
| **ForeignKey** | A link from one table to another (for example, each listing points to one seller). |
| **Primary key (pk / id)** | The unique number of each row, created automatically. |
| **Session** | How Django remembers you are logged in between requests (with a cookie). |
| **Middleware** | Code that runs on every request before and after the view (for sessions, login, CSRF protection...). |
| **Context** | The dictionary of data a view passes to a template. |
| **Decorator** | A line starting with `@` above a function that adds behavior. We use only Django's `@login_required`. |
| **Admin site** | A ready-made management website Django builds from our models, at `/admin/`. |
| **Static files** | CSS, JS, images that belong to the site. We load Bootstrap from a CDN instead. |
| **Media files** | Files that users upload, like listing photos. Saved in `media/`. |
| **CDN** | A public server that hosts common files like Bootstrap so we do not need to copy them. |
| **CSRF** | A type of attack. Django blocks it with a hidden token in every form (`{% csrf_token %}`). |

## 2.5 Template language in 30 seconds

```html
{{ listing.title }}                 show a value
{% if user.is_authenticated %}      if statement
{% for c in categories %}           loop
{% url 'detail' listing.pk %}       build a URL from its name
{% extends 'base.html' %}           reuse the main layout
{% block content %}...{% endblock %} fill a hole in the layout
{% include 'reviews/section.html' %} paste another small template here
{{ listing.description|linebreaksbr }}  a "filter" that changes how a value is shown
```

Now you know enough to read the rest.
