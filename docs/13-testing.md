# 13. Testing

## 13.1 What is an automated test?

A small program that uses the app like a user would and checks the result. If someone later breaks a feature, the test fails and tells us. Django's test tool creates a **separate empty database** for tests, so real data is never touched.

Run all tests:

```bash
.venv/bin/python manage.py test
```

Result at the time of writing: **16 tests, all pass.**

Run one test:

```bash
.venv/bin/python manage.py test catalog.tests.ListingTest.test_post_then_approve
```

(Check the class name inside each `tests.py`.)

## 13.2 How a test looks

Tests use `self.client`, a fake browser:

```python
response = self.client.get('/no-such-page/')
self.assertEqual(response.status_code, 404)
self.assertContains(response, 'Page not found', status_code=404)
```

## 13.3 Every test and what it proves

### accounts/tests.py

| Test | Proves |
|------|--------|
| `test_register_needs_matching_record` | A wrong phone number does not register the student |
| `test_register_once` | Registration works once; a second try says "already registered" |
| `test_login_required_shows_401_with_next` | Opening `/new/` logged out shows the 401 page with `next=/new/` |
| `test_missing_page_shows_custom_404` | Unknown URLs show our 404 page |
| `test_login_goes_back_to_next` | After login you return to `next`, but `next=http://evil.com/` is ignored |

### catalog/tests.py

| Test | Proves |
|------|--------|
| `test_post_then_approve` | A new listing is hidden until approved, then appears |
| `test_rent_without_period_rejected` | A rent listing without a period is refused |
| `test_donate_is_free_and_marked_donated` | Donate listings show "Free" and "Donated" |
| `test_contact_hidden_until_request_then_seller_accepts` | Phone is hidden until a request, then shown; seller can accept |
| `test_cannot_accept_after_sold` | Once one request is accepted, a second one cannot be accepted |

### inbox/tests.py

| Test | Proves |
|------|--------|
| `test_buyer_and_seller_chat` | Buyer and seller can both send and see messages |
| `test_outsider_cannot_open_or_post` | A third student gets 404 for someone else's chat |
| `test_anonymous_gets_401` | Logged-out users cannot open the inbox |
| `test_new_messages_returns_only_newer_ones` | The live-chat endpoint returns only messages after `after` |

### reviews/tests.py

| Test | Proves |
|------|--------|
| `test_unverified_buyer_cannot_review` | Someone without an accepted request cannot review |
| `test_verified_review_needs_admin_approval` | A verified buyer's review is hidden until the admin approves it |

## 13.4 Why test these and not everything?

We tested the **rules that matter most**: who can see what, who can do what, and the trickiest logic (registration matching, moderation, chat privacy, review verification). Simple pages that only display data are not worth separate tests.
