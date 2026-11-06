# Member 3 – Guests and Event Participation

**Apps you own:** `guests`
**Extra shared task:** API documentation (drf-spectacular, Postman collection)
**Depends on:** Member 1's `User` and `Event` (Day 3).
**Others depend on you:** Member 4 references `Guest` for gift distribution; Member 1 uses `Event.guests` for access checks.

Read `00_SHARED_CONVENTIONS.md` first.

---

## What you build

### A. Guest lists and invitations
**Models**
- `Guest`: event (`related_name="guests"`), `name`, `email`, `phone`, `user` (FK to User, nullable), `added_by`, `is_approved` (client approval), unique (event, email)
- `Invitation`: guest, `token` (unique, random), `sent_at`, `status` (CREATED / SENT / ACCEPTED / DECLINED), `expires_at`

**Endpoints**
| Method | Route | Who |
|--------|-------|-----|
| CRUD | `/events/<id>/guests/` | admin all; client of that event manages own list |
| POST | `/events/<id>/guests/bulk/` | admin or client (CSV or JSON list; return per-row errors) |
| POST | `/events/<id>/guests/<gid>/invite/` | admin (creates invitation; email sending can be console backend for now) |
| POST | `/events/<id>/invitations/send-all/` | admin |
| GET | `/me/invitations/` | guest |

### B. Account linking and RSVP
**Model**
- `RSVP`: guest, `response` (YES / NO / MAYBE), `plus_ones` (int), `note`, `responded_at`

**Endpoints**
| Method | Route | Who |
|--------|-------|-----|
| GET | `/invitations/<token>/` | public (shows event name and date only) |
| POST | `/invitations/<token>/link-account/` | logged-in GUEST: links `guest.user` if the email matches |
| POST | `/invitations/<token>/rsvp/` | linked guest |

**Validation**
- Token expired or already used by a different account → 400.
- Email on invitation must match the logged-in user's email (or admin override).
- RSVP can be changed until event start date, then locked.

### C. Check-in
**Model**
- `CheckIn`: guest (OneToOne), `checked_in_at`, `checked_in_by` (employee/admin), `method` (MANUAL / QR)

**Endpoints**
| Method | Route | Who |
|--------|-------|-----|
| POST | `/events/<id>/checkin/` | assigned employee or admin (body: guest id or token) |
| GET | `/events/<id>/checkin/summary/` | admin, assigned employees (checked in vs expected) |

**Validation:** second check-in for the same guest returns **409 Conflict** with the earlier time (use DB unique constraint plus `get_or_create` inside a transaction). Only ACCEPTED/approved guests can check in (admin can override with a reason).

### D. Program, timings, map
**Models**
- `ProgramItem`: event, title, description, `start_time`, `end_time`, `location_note`, `order`
- `EventMap`: event (OneToOne), `image` or `map_url`, `description`, `venue_address`, `latitude`, `longitude`

**Endpoints**
| Method | Route | Who |
|--------|-------|-----|
| CRUD | `/events/<id>/program/` | admin write; any user with event access reads |
| GET/PUT | `/events/<id>/map/` | admin write; invited guests, client, employees read |

**Validation:** `end_time > start_time`; program item times must fall within the event dates; map is hidden from guests who have not RSVP'd YES (decide with team and document).

### E. Ratings and feedback
**Model**
- `Feedback`: event, `author`, `author_role` (CLIENT / GUEST), `rating` (1–5), `comment`, unique (event, author)

**Endpoints**
| Method | Route | Who |
|--------|-------|-----|
| POST | `/events/<id>/feedback/` | client or checked-in guest, only after event is COMPLETED |
| GET | `/events/<id>/feedback/` | admin all; client sees all; guest sees own |
| GET | `/events/<id>/feedback/summary/` | admin: average rating, count |

### F. Extra shared task: API documentation
- [ ] Configure drf-spectacular: `/api/schema/`, `/api/docs/`
- [ ] Add JWT security scheme to the docs
- [ ] Export a Postman collection into `docs/` and update it after integration
- [ ] Remind members to add `@extend_schema` to every route

---

## To-do list

### Phase 1 – Days 2–3
- [ ] Create `guests` app and models (migrations on your branch)
- [ ] Set up Swagger and the docs URLs on a small branch, merge once Member 1's skeleton exists

### Phase 2 – Days 4–12
- [ ] Guest CRUD and bulk import with per-row validation
- [ ] Invitation creation, sending, token lookup
- [ ] Account linking and RSVP
- [ ] Check-in with duplicate prevention and summary
- [ ] Program items and map endpoints
- [ ] Feedback with the "only after completion" rule
- [ ] Tests: success, invalid input, unauthorized (include double check-in, expired token, guest from another event)
- [ ] Postman collection first version
- [ ] Review Member 4's pull requests

### Phase 3 – Days 13–15
- [ ] Confirm Member 4 can reference `Guest` and `CheckIn`
- [ ] Update Postman and docs after all modules merge
- [ ] Demo step: invite → RSVP → check-in → feedback

## Definition of done
- No guest can see or act on another event's data, double check-in is impossible, and all your routes appear in Swagger.