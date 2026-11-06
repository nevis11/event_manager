# Member 1 – Accounts and Event Foundation

**Apps you own:** `accounts`, `events`
**Why it matters:** every other member links to your `User` and `Event` models. Your first priority is merging them to `develop` by **Day 3**.

Read `00_SHARED_CONVENTIONS.md` first.

---

## What you build

### A. Accounts (`accounts`)
**Models**
- `TimeStampedModel` (abstract: created_at, updated_at)
- `User(AbstractUser)`: `email` (unique, used for login), `role` (ADMIN / CLIENT / EMPLOYEE / GUEST), `phone`, `is_active`
- `Profile`: OneToOne to User, `address`, `photo`, `bio`

**Endpoints**
| Method | Route | Who | Purpose |
|--------|-------|-----|---------|
| POST | `/auth/register/` | public | Registers CLIENT or GUEST only (never ADMIN/EMPLOYEE) |
| POST | `/auth/login/` | public | Returns access and refresh tokens |
| POST | `/auth/logout/` | logged in | Blacklists refresh token |
| POST | `/auth/token/refresh/` | public | Refresh access token |
| GET/PUT | `/auth/me/` | logged in | Own user and profile |
| POST | `/users/` | admin | Admin creates EMPLOYEE / ADMIN accounts |
| GET | `/users/?role=` | admin | List and filter users |
| PATCH | `/users/<id>/` | admin | Activate / deactivate, change role |

**Validation:** unique email, password strength, role cannot be changed by the user themselves.

### B. Permissions (`accounts/permissions.py`)
Implement exactly the helpers listed in the shared conventions: `IsAdmin`, `IsClient`, `IsEmployee`, `IsGuest`, `IsAdminOrReadOnly`, `user_can_access_event()`, `EventAccessPermission`.
`user_can_access_event` uses the related names `event.assignments` and `event.guests`. Until Members 2 and 3 merge, guard with `hasattr` or lazy imports so it does not crash.

### C. Event requests and events (`events`)
**Models**
- `EventRequest`: client, title, event_type, desired_date, venue_preference, expected_guests, budget_estimate, notes, status (PENDING / APPROVED / REJECTED), reviewed_by, review_note, `event` (OneToOne, null until approved)
- `Event`: title, event_type, client, description, start_date, end_date, venue, expected_guests, agreed_budget, status (APPROVED / PLANNING / ONGOING / COMPLETED / CANCELLED), created_by
- `EventRequirement`: event, item (e.g. chairs, stage), quantity, notes

**Endpoints**
| Method | Route | Who | Purpose |
|--------|-------|-----|---------|
| POST | `/event-requests/` | client | Submit request |
| GET | `/event-requests/` | client (own) / admin (all) | List |
| POST | `/event-requests/<id>/approve/` | admin | Approve and auto-create Event |
| POST | `/event-requests/<id>/reject/` | admin | Reject with review_note |
| CRUD | `/events/` | admin write; others read if `user_can_access_event` | Events |
| PATCH | `/events/<id>/status/` | admin | Move status (validate allowed transitions) |
| CRUD | `/events/<id>/requirements/` | admin write; client read own | Requirements |
| GET | `/events/?status=` | by access | Filter (upcoming, ongoing, completed) |

**Validation rules**
- `end_date >= start_date`; start_date cannot be in the past for new events.
- Allowed status flow: APPROVED → PLANNING → ONGOING → COMPLETED; CANCELLED from any except COMPLETED.
- Cannot approve or reject a request that is not PENDING.
- A client only sees their own requests and events.

---

## To-do list

### Phase 0 – Day 1
- [ ] Create the repo, Django project `config`, split settings (`base`, `dev`, `prod`), `.env.example`
- [ ] Add DRF, SimpleJWT, django-environ, drf-spectacular to `requirements.txt`
- [ ] Create `accounts` and `events` apps, abstract `TimeStampedModel`
- [ ] Add the other members to GitHub, protect `main` and `develop`

### Phase 1 – Days 2–3 (blocking for everyone)
- [ ] Custom `User` model, first migration (**do this before any other migration exists**)
- [ ] `Event` model and migration
- [ ] Permission helpers in `accounts/permissions.py`
- [ ] `accounts/tests/factories.py` (UserFactory by role, EventFactory)
- [ ] Merge to `develop` and announce in group chat

### Phase 2 – Days 4–12
- [ ] Register, login, logout, refresh, `me`
- [ ] Admin user management endpoints
- [ ] Profile model and update
- [ ] EventRequest create / list / approve / reject (approval creates Event in one transaction)
- [ ] Event CRUD with access filtering
- [ ] Status transitions with validation
- [ ] EventRequirement CRUD
- [ ] Tests: success, invalid input, unauthorized for every endpoint above
- [ ] Swagger docs complete for your routes
- [ ] Review Member 2's pull requests

### Phase 3 – Days 13–15
- [ ] Connect `user_can_access_event` to the real assignment and guest relations; test all four roles
- [ ] Fix integration issues reported by others
- [ ] Demo step: client request → approval → event

## Definition of done
- All endpoints documented in Swagger, tests pass, no endpoint accessible with the wrong role, and your two apps have clean migrations.