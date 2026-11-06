# Member 5 – Arrangements and Financial Dashboard

**Apps you own:** `arrangements`, `finance`
**Extra shared task:** final demo script and README update
**Depends on:** Member 1 (`User`, `Event`, `agreed_budget`), Member 2 (`EventAssignment`, `DamageReport`).

Read `00_SHARED_CONVENTIONS.md` first.

---

## What you build

### A. Manpower and material requests (`arrangements`)
**Models**
- `ManpowerRequest`: event, requested_by (employee/manager), `role_needed`, `count`, `needed_from`, `needed_to`, `reason`, `status` (PENDING / APPROVED / REJECTED / FULFILLED), `reviewed_by`, `review_note`
- `MaterialRequest`: event, requested_by, `item_name`, `quantity`, `estimated_cost`, `needed_by`, `reason`, `status` (same flow), `reviewed_by`, `review_note`

**Endpoints**
| Method | Route | Who |
|--------|-------|-----|
| POST/GET | `/events/<id>/manpower-requests/` | assigned employee creates; admin reads all |
| POST/GET | `/events/<id>/material-requests/` | same |
| POST | `/manpower-requests/<id>/approve/` or `/reject/` | admin |
| POST | `/material-requests/<id>/approve/` or `/reject/` | admin |
| POST | `/manpower-requests/<id>/fulfil/` | assigned employee, admin (marks FULFILLED) |

### B. Accommodation and emergency transport
**Models**
- `AccommodationRequest`: event, `for_user` or `for_guest` (nullable FK, one of them required), `requested_by`, `hotel_or_place`, `check_in`, `check_out`, `status` (PENDING / APPROVED / REJECTED / ARRANGED / COMPLETED), `cost` (nullable), `notes`
- `TransportRequest`: event, `for_user` / `for_guest`, `requested_by`, `pickup`, `dropoff`, `needed_at`, `is_emergency`, `status` (same flow), `vehicle_info`, `cost`, `notes`

**Endpoints**
| Method | Route | Who |
|--------|-------|-----|
| POST | `/events/<id>/accommodation/` and `/transport/` | client, guest (own), employee |
| GET | same | admin all; requester own; assigned employees read |
| POST | `.../<id>/approve/` / `/reject/` | admin |
| POST | `.../<id>/arrange/` | assigned employee (adds details) |
| POST | `.../<id>/complete/` | assigned employee, admin |

**Rules**
- Emergency transport (`is_emergency=True`) appears at the top of admin lists and notifies via a status flag (no real notifications needed).
- `check_out > check_in`; cannot approve twice; employees can only mark arranged or completed on **approved** requests.
- Approved costs feed into finance (see section C).

### C. Finance (`finance`)
**Models**
- `Payment`: event, `amount`, `paid_on`, `method` (CASH / BANK / CARD / UPI), `reference`, `received_by`, `note`
- `PaymentSchedule` (optional, simple): event, `due_date`, `amount`, `status` (DUE / PAID / OVERDUE)
- `Expense`: event, `category` (MATERIAL / MANPOWER / TRANSPORT / ACCOMMODATION / OTHER), `amount`, `spent_on`, `description`, `receipt` (file), `recorded_by`, optional links to the approved material, manpower, transport or accommodation request
- `DamageCharge`: event, `damage_report` (OneToOne to Member 2's `DamageReport`), `amount`, `charged_to_client` (bool), `note`, `created_by`

**Endpoints**
| Method | Route | Who |
|--------|-------|-----|
| CRUD | `/events/<id>/payments/` | admin write; client reads own event |
| CRUD | `/events/<id>/payment-schedule/` | admin write; client reads |
| CRUD | `/events/<id>/expenses/` | admin; assigned employee can add with receipt (needs admin approval flag) |
| POST | `/events/<id>/damage-charges/` | admin, only for APPROVED damage reports |
| GET | `/events/<id>/damage-charges/` | admin; client sees `charged_to_client=True` |
| GET | `/events/<id>/finance/summary/` | admin |
| GET | `/admin/dashboard/kpis/` | admin (all events, filterable by date range or event) |

**Validation**
- Amounts > 0, decimal only, no future `paid_on` dates.
- Receipt: pdf/jpg/png, max 5 MB.
- A `DamageCharge` can only be created for an APPROVED damage report with no existing charge, and `amount` defaults to `approved_cost` (admin may change with a note).
- Deleting a payment or expense: admin only, and soft delete (flag) so totals stay auditable.

**KPI formulas (document in code and tests)**
```
agreed_budget      = Event.agreed_budget
actual_spending    = sum(Expense.amount)            # include approved request costs once recorded as Expense
damage_charges     = sum(DamageCharge.amount where charged_to_client)
total_billed       = agreed_budget + damage_charges
amount_received    = sum(Payment.amount)
outstanding        = total_billed - amount_received
profit_estimate    = amount_received - actual_spending
budget_variance    = agreed_budget - actual_spending
```
The summary response returns every figure above plus an expense breakdown by category. Use `aggregate()`/`Sum()` in the database, not Python loops. Make sure totals stay correct when there are no records (return 0.00, not null).

### D. Extra shared task: final demo
- [ ] Write `docs/DEMO_SCRIPT.md`: the 12 steps of the final integration demo with the exact API calls
- [ ] Update the root `README.md` (Django setup, run, test commands)

---

## To-do list

### Phase 1 – Days 2–3
- [ ] Create `arrangements` and `finance` apps and models (migrations on your branch)
- [ ] Agree `DamageReport` to `DamageCharge` link and `agreed_budget` field name with Members 1 and 2
- [ ] Write the KPI formulas into a design note and share with the team

### Phase 2 – Days 4–12
- [ ] Manpower and material requests with approve / reject / fulfil
- [ ] Accommodation and transport requests with status flow and emergency priority
- [ ] Payments and payment schedule
- [ ] Expenses with receipt upload and links to requests
- [ ] DamageCharge creation linked to reviewed reports
- [ ] Finance summary per event and KPI dashboard
- [ ] Tests: success, invalid input, unauthorized (include charge on non-approved damage report, outstanding balance with and without payments, employee trying to approve)
- [ ] Swagger docs for your routes
- [ ] Review Member 1's pull requests

### Phase 3 – Days 13–15
- [ ] Verify KPI numbers against the seed data by hand
- [ ] Write the demo script and run the full integration demo with the team
- [ ] Update README

## Definition of done
- Every financial figure on the dashboard can be explained from stored records, damage charges stay linked to reviewed reports, and no non-admin can approve requests or edit finance data.