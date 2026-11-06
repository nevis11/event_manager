# Member 2 – Employee Management, Tasks and Damage Reports

**Apps you own:** `employees`
**Extra shared task:** Docker and CI (see section D)
**Depends on:** Member 1's `User` and `Event` (Day 3). Until then, write models and tests against stubs.
**Others depend on you:** Member 1 uses `Event.assignments`; Member 5 uses `DamageReport` for charges.

Read `00_SHARED_CONVENTIONS.md` first.

---

## What you build

### A. Employee details and availability
**Models**
- `EmployeeProfile`: OneToOne `user` (role must be EMPLOYEE), `designation`, `skills` (text or JSON), `phone`, `is_active`
- `Availability`: employee, `date`, `is_available`, `note` (unique per employee and date)

**Endpoints**
| Method | Route | Who |
|--------|-------|-----|
| CRUD | `/employees/` | admin write; employee reads own |
| GET/POST/PATCH | `/employees/<id>/availability/` | employee edits own; admin reads |
| GET | `/employees/available/?date=` | admin (find free staff) |

### B. Assignments, tasks, communication
**Models**
- `EventAssignment`: event (`related_name="assignments"`), employee, `is_manager` (bool), `role_note`, unique (event, employee)
- `Task`: event, title, description, assigned_to (employee), assigned_by, due_date, status (PENDING / ACTIVE / FINISHED), priority
- `Comment`: author, event, optional `task`, `body` (event discussion if no task, task comment otherwise)

**Endpoints**
| Method | Route | Who |
|--------|-------|-----|
| POST/GET/DELETE | `/events/<id>/assignments/` | admin |
| POST | `/events/<id>/assignments/<aid>/make-manager/` | admin |
| CRUD | `/events/<id>/tasks/` | admin or event manager create; assignee reads |
| PATCH | `/tasks/<id>/progress/` | assignee only (status change) |
| GET | `/me/tasks/?status=` | employee |
| GET/POST | `/events/<id>/discussion/` | assigned employees, admin, client optional read |
| GET/POST | `/tasks/<id>/comments/` | assignee, creator, admin |

**Validation**
- Cannot assign an employee marked unavailable for the event dates (return a clear error).
- Cannot assign the same employee twice to one event.
- Task can only go PENDING → ACTIVE → FINISHED (no skipping backward unless admin).
- Only an employee assigned to the event can be given a task for that event.
- Only an assigned manager or admin can create tasks, and only the assignee can update progress.

### C. Damage reports
**Models**
- `DamageReport`: event (`related_name="damage_reports"`), reported_by (employee), `description`, `item_name`, `quantity`, `estimated_cost` (nullable), `photo`, `status` (SUBMITTED / UNDER_REVIEW / APPROVED / REJECTED), `reviewed_by`, `review_note`, `approved_cost` (null until approved), `visible_to_client` (bool, default False)

**Endpoints**
| Method | Route | Who |
|--------|-------|-----|
| POST | `/events/<id>/damage-reports/` | assigned employee |
| GET | `/events/<id>/damage-reports/` | admin all; employee own; client only `visible_to_client=True` and APPROVED |
| POST | `/damage-reports/<id>/review/` | admin: approve (sets `approved_cost`, `visible_to_client`) or reject with note |

**Contract with Member 5:** once a report is APPROVED, Member 5 can create a `DamageCharge` with a FK to your report. You must not allow editing `approved_cost` or deleting an approved report that has a charge. Expose a read-only `has_charge` field (Member 5 sets this relation; coordinate).

### D. Extra shared task: DevOps
- [ ] `Dockerfile` for Django and `docker-compose.yml` (web + postgres)
- [ ] `.dockerignore`
- [ ] GitHub Actions workflow: install, run `pytest` on every pull request

---

## To-do list

### Phase 1 – Days 2–3
- [ ] Create `employees` app and write models (migrations on your feature branch only)
- [ ] Write the Docker and CI files (independent of other work)
- [ ] Agree `DamageReport` fields with Member 5

### Phase 2 – Days 4–12
- [ ] EmployeeProfile CRUD (creating an employee user is done by admin via Member 1's `/users/`; link it here)
- [ ] Availability endpoints and available-by-date search
- [ ] EventAssignment with manager flag and conflict validation
- [ ] Task CRUD and progress updates with transition rules
- [ ] Event discussion and task comments
- [ ] DamageReport submit, list with visibility rules, review
- [ ] Tests: success, invalid input, unauthorized for each feature (include "employee not assigned to this event" and "client sees hidden report" cases)
- [ ] Swagger docs for your routes
- [ ] Review Member 3's pull requests

### Phase 3 – Days 13–15
- [ ] Verify `user_can_access_event` works for employees
- [ ] Confirm Member 5's charges link correctly and approved reports are protected
- [ ] Demo step: assign staff → tasks → progress → damage report → review

## Definition of done
- Docker setup runs the project with one command, CI passes, and every employee workflow works with correct permissions.