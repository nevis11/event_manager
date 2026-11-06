# Shared Conventions (everyone reads this first)

Stack: **Python 3.12, Django 5, Django REST Framework, SimpleJWT (with blacklist for logout), PostgreSQL, pytest-django, drf-spectacular (API docs).**

## 1. Django apps and owners
| App | Owner | Contains |
|-----|-------|----------|
| `accounts` | Member 1 | User, Profile, roles, permissions |
| `events` | Member 1 | EventRequest, Event, EventRequirement |
| `employees` | Member 2 | EmployeeProfile, Availability, EventAssignment, Task, Comment, DamageReport |
| `guests` | Member 3 | Guest, Invitation, CheckIn, ProgramItem, EventMap, Feedback |
| `gifts` | Member 4 | IncomingGift, ReturnGiftItem, Distribution, EventPhoto, PortfolioItem |
| `arrangements` | Member 5 | Manpower/Material requests, Accommodation, Transport |
| `finance` | Member 5 | Payment, Expense, DamageCharge, KPI dashboard |

## 2. Roles
`ADMIN`, `CLIENT`, `EMPLOYEE`, `GUEST` stored in `User.role`.
"Manager" = an employee assigned to an event with `EventAssignment.is_manager=True` (not a separate role).

## 3. Contract between apps (do not rename without telling everyone)
- Every event-linked model has `event = ForeignKey("events.Event", on_delete=CASCADE, related_name="<plural>")`.
- Related names that others rely on:
  - `Event.assignments` (EventAssignment, Member 2) with `.employee.user` and `.is_manager`
  - `Event.guests` (Guest, Member 3) with `.user` (nullable until linked)
  - `Event.damage_reports` (DamageReport, Member 2)
  - `Event.photos` (EventPhoto, Member 4)
  - `Event.payments`, `Event.expenses` (Member 5)
- User FK: `settings.AUTH_USER_MODEL`.
- Money: `DecimalField(max_digits=12, decimal_places=2)`. Never float.
- Timestamps: every model has `created_at`, `updated_at` (abstract `TimeStampedModel` in `accounts/models.py`, Member 1 provides it on Day 1).

## 4. Permission helpers (provided by Member 1 in `accounts/permissions.py`)
```
IsAdmin, IsClient, IsEmployee, IsGuest            # role checks
IsAdminOrReadOnly
user_can_access_event(user, event) -> bool         # admin: all; client: own events;
                                                   # employee: assigned; guest: invited
EventAccessPermission                              # object-level, uses the function above
```
Use these. Do not write your own role checks.

## 5. URL and response conventions
- Prefix: `/api/v1/`. Event-linked routes: `/api/v1/events/<event_id>/<resource>/`.
- Use DRF `ModelViewSet` / `ViewSet` + routers; one `urls.py` per app, included from `config/urls.py`.
- Errors use DRF defaults (`400` validation, `401` not logged in, `403` no permission, `404` not found).
- Pagination: `PageNumberPagination`, page size 20.
- Every route needs `@extend_schema` or serializer docs so it appears in Swagger (`/api/docs/`).

## 6. Git workflow
- Branches: `main` (stable) ← `develop` ← `feature/<app>-<short-name>`.
- Commit style: `feat(guests): add RSVP endpoint`.
- Pull requests need **1 approval** from the assigned reviewer (see review ring below).
- Never commit `.env`, `db.sqlite3`, or migration conflicts. If a migration conflict happens, the person who merged second runs `makemigrations --merge`.
- Each person creates only their own app's migrations.

**Review ring:** M1 reviews M2 → M2 reviews M3 → M3 reviews M4 → M4 reviews M5 → M5 reviews M1.

## 7. Testing rules (every member, every feature)
Per feature write at least:
1. one **successful workflow** test,
2. one **invalid input** test,
3. one **unauthorized access** test (wrong role and not logged in).

Run: `pytest`. Test files: `<app>/tests/test_<feature>.py`. Use factory helpers from `accounts/tests/factories.py` (Member 1 provides).

## 7b. Timeline overview
| Phase | Days | What |
|-------|------|------|
| 0 | 1 | Kick-off, conventions agreed, repo and environment ready |
| 1 | 2–3 | Member 1 merges User, Event, permissions. Others finish models (migrations on their branches) |
| 2 | 4–12 | All five build features in parallel |
| 3 | 13–15 | Integration, cross-module fixes, one complete event demo |
| 4 | 16 | Compare progress, redistribute leftover work |

## 8. Extra shared tasks (so nothing is unowned)
| Task | Owner |
|------|-------|
| Project skeleton, settings split, `.env.example`, base abstract models | Member 1 |
| Dockerfile, docker-compose (Django + Postgres), GitHub Actions running pytest | Member 2 |
| drf-spectacular setup, `/api/docs/`, Postman collection export | Member 3 |
| `seed_demo` management command (demo data for the final event demo) | Member 4 |
| Final demo script and README update | Member 5 |

## 9. Final integration demo (all five)
One event goes through: client request → admin approval → event created → employees assigned → tasks done → guests invited and RSVP → check-in → gifts collected and distributed → photos uploaded → damage reported and charged → expenses and payments recorded → KPI dashboard shows correct totals.