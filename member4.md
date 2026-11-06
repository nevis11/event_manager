# Member 4 – Gifts, Photos and Portfolio

**Apps you own:** `gifts`
**Extra shared task:** `seed_demo` management command
**Depends on:** Member 1 (`User`, `Event`), Member 3 (`Guest`, from about Day 6; use a stub until then).

Read `00_SHARED_CONVENTIONS.md` first.

---

## What you build

### A. Incoming gifts (gifts given by guests)
**Model**
- `IncomingGift`: event, `giver_name`, `guest` (FK to Guest, nullable), `description`, `estimated_value` (optional), `collected_by` (employee), `collected_at`, `notes`

**Endpoints**
| Method | Route | Who |
|--------|-------|-----|
| POST | `/events/<id>/incoming-gifts/` | assigned employee, admin (`collected_by` = current user, `collected_at` = now by default) |
| GET | `/events/<id>/incoming-gifts/` | admin, assigned employees, client of the event |
| PATCH/DELETE | `/incoming-gifts/<id>/` | admin only |

### B. Return gifts supplied by the client
**Model**
- `ReturnGiftItem`: event, `item_name`, `quantity_supplied` (>0), `supplied_by_client`, `notes`, `photo` (optional)

**Endpoints**
| Method | Route | Who |
|--------|-------|-----|
| CRUD | `/events/<id>/return-gifts/` | client adds or edits own event items; admin all; employees read |

### C. Distribution records
**Model**
- `Distribution`: return_gift_item, `guest` (FK to Guest), `quantity`, `distributed_by` (employee), `distributed_at`

**Endpoints**
| Method | Route | Who |
|--------|-------|-----|
| POST | `/events/<id>/distributions/` | assigned employee, admin |
| GET | `/events/<id>/distributions/` | admin, client, assigned employees |
| GET | `/events/<id>/distributions/by-guest/<gid>/` | admin, employees |

**Validation**
- `guest` must belong to the same event and have been **checked in** (use Member 3's `CheckIn`; admin override with reason).
- `quantity` > 0 and cannot exceed remaining stock.
- Use `transaction.atomic()` with `select_for_update()` on the item so two employees cannot over-distribute at the same time.
- Optional rule (agree with team): one distribution per guest per item.

### D. Remaining stock
- Do not store stock manually. Compute `remaining = quantity_supplied - sum(distributions)`.
- `GET /events/<id>/return-gifts/stock/` returns, per item: supplied, distributed, remaining.
- Add a `low_stock` flag when remaining is under 10 percent.
- Reject edits that reduce `quantity_supplied` below the already distributed amount.

### E. Event photos and portfolio
**Models**
- `EventPhoto`: event (`related_name="photos"`), `image`, `caption`, `uploaded_by`, `uploaded_at`, `is_public_sample` (bool, default False)
- `PortfolioItem`: photo (OneToOne), `title`, `description`, `event_type`, `published_by`, `published_at`, `is_published`

**Endpoints**
| Method | Route | Who |
|--------|-------|-----|
| POST | `/events/<id>/photos/` | assigned employee, admin (validate image type, size limit 5 MB) |
| GET | `/events/<id>/photos/` | admin, client (own event), assigned employees |
| DELETE | `/photos/<id>/` | uploader or admin |
| POST | `/photos/<id>/publish/` | admin (creates PortfolioItem) |
| POST | `/portfolio/<id>/unpublish/` | admin |
| GET | `/portfolio/` | **public** (no login; only published items; filter by `event_type`) |

**Validation:** only jpg/png/webp, max 5 MB; a photo from event X cannot be published without admin action; unpublished items must never appear on the public route.

### F. Extra shared task: demo data
Create `python manage.py seed_demo` (put it in `accounts/management/commands/` or a small `core` app agreed with Member 1). It should create: 1 admin, 2 clients, 5 employees, 1 completed and 1 ongoing event, guests, program items, gifts, photos (small placeholder images), payments and expenses. Run it last after all models exist, but build the skeleton early.

---

## To-do list

### Phase 1 – Days 2–3
- [ ] Create `gifts` app and models (migrations on your branch)
- [ ] Configure media storage (`MEDIA_ROOT`, `MEDIA_URL`) with Member 1
- [ ] Draft the `seed_demo` skeleton

### Phase 2 – Days 4–12
- [ ] IncomingGift endpoints
- [ ] ReturnGiftItem endpoints and quantity validation
- [ ] Distribution with stock check and concurrency lock
- [ ] Stock summary endpoint
- [ ] Photo upload with file validation
- [ ] Publish and unpublish portfolio, public portfolio route
- [ ] Tests: success, invalid input, unauthorized (include over-distribution, guest not checked in, public route not leaking unpublished photos)
- [ ] Swagger docs for your routes
- [ ] Review Member 5's pull requests

### Phase 3 – Days 13–15
- [ ] Connect `Distribution` to Member 3's real `Guest` and `CheckIn`
- [ ] Finish and test `seed_demo` on a fresh database
- [ ] Demo step: collect gifts → distribute return gifts → upload and publish photos

## Definition of done
- Stock can never go negative, only checked-in guests receive gifts, and the public portfolio shows only published items.