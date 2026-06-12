# PRODUCT REQUIREMENTS DOCUMENT (PRD) — ExpenseSnap

> **Version:** 2.0 | **Date:** 2026-06-12

---

## 1. Problem Statement

* **The Issue:** Freelancers track business expenses manually using physical notebooks — a process that is error-prone, lacks visual insights, offers no spending guardrails, and makes tax reporting tedious. As their business grows, additional pain points emerge: repetitive data entry for recurring costs, no month-over-month trend visibility, inability to digitize paper receipts quickly, and no way to split shared costs with a co-founder or partner without resorting to spreadsheets.
* **Target Users:** Freelancers, independent contractors, solopreneurs, and 2-person micro-teams.
* **Impact:**

| Pain Point | Consequence |
|---|---|
| Manual notebook tracking | Lost records, inaccurate calculations, missed tax deductions |
| No real-time spending visibility | Budget overruns discovered too late |
| Repetitive entry of fixed costs | Wasted time; forgotten entries |
| No trend analysis across months | Gut-based financial decisions |
| No receipt digitization | Paper clutter; slow expense logging |
| No shared expense tracking | Partners resort to separate spreadsheets; settlement disputes |

---

## 2. Solution Overview

* **Value Prop:** A clean, lightweight expense tracking web application that lets freelancers instantly log expenses (manually or via receipt scan), set budget guardrails, automate recurring entries, visualize month-over-month trends, split shared costs with a partner, and export all records to CSV.

### Feature Matrix

| # | Feature | Priority | Category |
|---|---|---|---|
| F0a | Expense Logging (CRUD) | P0 | Core |
| F0b | Monthly Summary & Bar Chart | P0 | Core |
| F0c | CSV Export | P0 | Core |
| F1 | OCR Receipt Scanner | P0 | Advanced |
| F2 | Budget Limit Alerts | P0 | Advanced |
| F3 | Recurring Expense Auto-Entry | P1 | Advanced |
| F4 | Month-over-Month Comparison Chart | P1 | Advanced |
| F5 | Shared Expense Mode (2-User Split) | P2 | Advanced |

### Core Features (F0)

* **Expense Logging (CRUD):** Add, view, edit, and delete expenses containing amount, category, and date.
* **Monthly Summary & Chart:** Real-time monthly summary showing total expenses grouped by category, alongside a dynamic bar chart.
* **CSV Export:** One-click download of the entire expense log in standard CSV format.

### Advanced Features (F1–F5)

* **OCR Receipt Scanner:** Capture/upload a receipt photo → auto-extract amount, vendor, date → suggest category → user reviews & saves.
* **Budget Limit Alerts:** Set monthly soft/hard caps per category; real-time visual alerts at 80% and 100% thresholds.
* **Recurring Expense Auto-Entry:** Mark expenses as recurring (Daily/Weekly/Monthly) with a billing date; system auto-logs entries on schedule.
* **MoM Comparison Chart:** Dedicated analytics tab with bar/line chart comparing spending across months, filterable by category.
* **Shared Expense Mode:** Invite one peer via unique link/code to a shared ledger with 50/50 split tracking, settlement summary, and separate personal ledgers.

### Architecture Considerations

- **Auth:** F5 mandates user authentication; lightweight auth (email magic-link or OAuth) recommended as a platform prerequisite for F2–F5.
- **Backend:** F3 (server-side scheduler) and F5 (shared ledger) require a persistent backend, upgrading the MVP local-storage-only approach.
- **New Dependencies:** OCR API client (Google Vision / Tesseract.js), charting library with multi-dataset support, real-time pub/sub for F5.

### Out of Scope

* Integration with external banking APIs.
* Multi-currency support (single currency assumed; currency code field stored for forward compatibility).
* AI-powered auto-categorization beyond vendor→category mapping from OCR.
* Team/group expense splitting beyond 2 users.

---

## 3. User Flow

### F0 — Core Expense Tracking

```
1. Freelancer incurs a business expense (e.g., client meeting coffee, software subscription).
2. Opens ExpenseSnap dashboard.
3. Clicks "Add Expense" → enters amount, selects/inputs category, selects date.
4. System validates inputs, saves entry, dynamically updates monthly summary and bar chart.
5. User clicks "Export to CSV" → CSV file downloads to local device.
```

### F1 — OCR Receipt Scanner

```
User taps "Scan Receipt"
  → Camera opens OR file picker appears
  → User captures/selects image
  → Upload spinner shown (skeleton form visible behind, NO layout shift)
  → OCR processes image → returns {amount, vendor, date}
  → Auto-fills expense form: Amount (extracted), Category (suggested via vendor→category map), Date (extracted or today)
  → User reviews, edits if needed, taps "Save"
  → Expense saved → Dashboard updates
```

**UI/UX Constraints:**
- Processing overlay must be a translucent sheet OVER the form — form fields remain in place, no vertical reflow.
- Keyboard dismiss on scan initiation; re-summon only when user taps an editable field post-parse.
- Confidence indicator per field (green check = high confidence, yellow = review suggested).

### F2 — Budget Limit Alerts

```
User navigates to Settings → Budget Limits
  → Taps "Add Limit" → Selects Category, enters Amount, chooses Limit Type (Soft/Hard)
  → Saves limit
  → On each expense save:
      System recalculates category total for current month
      → If total ≥ 80% of limit → Soft alert (amber banner + optional push notification)
      → If total ≥ 100% of limit → Hard alert (red banner + blocking confirmation dialog for Hard limits)
  → User can still override Hard limit via explicit "Proceed Anyway" confirmation
```

### F3 — Recurring Expense Auto-Entry

```
User adds/edits an expense → Toggles "Make Recurring"
  → Selects frequency: Daily | Weekly | Monthly
  → Sets billing date (day-of-month for Monthly, day-of-week for Weekly)
  → Sets optional end date or "Until cancelled"
  → Saves → Recurring rule created
  → On scheduled date:
      System auto-creates expense entry (source: "auto")
      → Push notification: "Recurring expense ₹X for [Category] logged"
  → User can view/edit/cancel recurring rules from Settings → Recurring Expenses
```

### F4 — MoM Comparison Chart

```
User navigates to Analytics tab
  → Default view: Bar chart comparing current month vs. previous month (total spend)
  → User can:
      Toggle chart type (Bar ↔ Line)
      Select date range (last 3 / 6 / 12 months)
      Filter by category (multi-select dropdown)
  → Chart animates on filter change
  → Tap on a bar/point → Tooltip shows breakdown: total, top 3 categories, % change from prior month
```

### F5 — Shared Expense Mode

```
User A taps "Create Shared Space"
  → System generates unique invite code/link
  → User A shares link with User B
  → User B opens link → Auth prompt (sign-up/login) → Joins shared space
  → Shared Space view:
      Combined ledger (all shared expenses from both users)
      "Who owes whom" settlement summary (running 50/50 balance)
      Combined monthly totals chart
  → Either user can log an expense to the shared ledger (tagged with payer)
  → Personal ledger remains completely separate and private
  → Either user can leave the shared space (historical data preserved, read-only)
```

---

## 4. API Design

### F0 — Core Expense CRUD

**Create Expense**
```
POST /api/v1/expenses
Body:
{
  "amount": 15.00,
  "category": "Meals",
  "date": "2026-06-10"
}

Response 200:
{
  "status": "success",
  "data": {
    "id": "exp_001",
    "amount": 15.00,
    "category": "Meals",
    "date": "2026-06-10"
  }
}
```

**List Expenses**
```
GET /api/v1/expenses

Response 200:
{
  "status": "success",
  "data": [
    {
      "id": "exp_001",
      "amount": 15.00,
      "category": "Meals",
      "date": "2026-06-10"
    }
  ]
}
```

**Delete Expense**
```
DELETE /api/v1/expenses/{id}

Response 200:
{
  "status": "success",
  "data": { "id": "exp_001" }
}
```

---

### F1 — OCR Receipt Scanner

```
POST /api/v2/receipts/scan
Content-Type: multipart/form-data
Body: { image: <file> }

Response 200:
{
  "status": "success",
  "data": {
    "parsed": {
      "amount": 42.50,
      "vendor": "Starbucks",
      "date": "2026-06-10",
      "confidence": { "amount": 0.95, "vendor": 0.88, "date": 0.92 }
    },
    "suggested_category": "Meals",
    "raw_text": "..."
  }
}

Response 422:
{
  "status": "error",
  "error": { "code": "OCR_PARSE_FAILED", "message": "Could not extract data from image." }
}
```

### F2 — Budget Limit Alerts

```
POST /api/v2/budgets
Body:
{
  "category": "Marketing",
  "limit_amount": 500.00,
  "limit_type": "hard",
  "month": "2026-06"
}

Response 200:
{
  "status": "success",
  "data": {
    "id": "bgt_001",
    "category": "Marketing",
    "limit_amount": 500.00,
    "limit_type": "hard",
    "current_spend": 320.00,
    "utilization_pct": 64
  }
}

GET /api/v2/budgets?month=2026-06
→ Returns array of all budget limits with current utilization.

DELETE /api/v2/budgets/{id}
```

### F3 — Recurring Expense Auto-Entry

```
POST /api/v2/recurring
Body:
{
  "amount": 29.99,
  "category": "Software",
  "frequency": "monthly",
  "billing_day": 15,
  "end_date": null
}

Response 200:
{
  "status": "success",
  "data": {
    "id": "rec_001",
    "amount": 29.99,
    "category": "Software",
    "frequency": "monthly",
    "billing_day": 15,
    "next_run": "2026-07-15",
    "status": "active"
  }
}

GET /api/v2/recurring → List all rules
PATCH /api/v2/recurring/{id} → Update rule (amount, frequency, pause/resume)
DELETE /api/v2/recurring/{id} → Cancel rule
```

### F4 — MoM Comparison Chart

```
GET /api/v2/analytics/mom?months=6&category=Marketing,Meals

Response 200:
{
  "status": "success",
  "data": {
    "months": [
      {
        "month": "2026-06",
        "total": 1250.00,
        "by_category": { "Marketing": 500, "Meals": 750 }
      },
      {
        "month": "2026-05",
        "total": 980.00,
        "by_category": { "Marketing": 430, "Meals": 550 }
      }
    ]
  }
}
```

### F5 — Shared Expense Mode

```
POST /api/v2/shared-spaces
Response 200:
{
  "status": "success",
  "data": {
    "id": "space_001",
    "invite_code": "XKCD42",
    "invite_link": "https://app.expensesnap.io/join/XKCD42",
    "owner_id": "user_001"
  }
}

POST /api/v2/shared-spaces/join
Body: { "invite_code": "XKCD42" }

POST /api/v2/shared-spaces/{id}/expenses
Body:
{
  "amount": 60.00,
  "category": "Meals",
  "date": "2026-06-10",
  "paid_by": "user_001"
}

GET /api/v2/shared-spaces/{id}/settlement
Response 200:
{
  "status": "success",
  "data": {
    "total_shared": 500.00,
    "user_001_paid": 350.00,
    "user_002_paid": 150.00,
    "settlement": { "user_002_owes_user_001": 100.00 }
  }
}

DELETE /api/v2/shared-spaces/{id}/members/{user_id} → Leave space
```

---

## 5. Edge Cases & Error Handling

### F0 — Core Expense Tracking

| Scenario | Handling |
|---|---|
| Empty/Invalid Inputs | UI blocks submission; highlights invalid fields with red border |
| No Expenses Recorded Yet | Display empty state placeholder: "No expenses logged yet. Add your first expense above!" and blank chart |
| Very Large Amounts (e.g., billions) | Validate and cap at $999,999.99 per transaction to prevent layout overflow |
| Special Characters in Category Name | Sanitize inputs to prevent XSS and formatting issues |

### F1 — OCR Receipt Scanner

| Scenario | Handling |
|---|---|
| Blurry / unreadable image | Return `OCR_PARSE_FAILED`; show "Could not read receipt. Try again with better lighting." |
| Partial parse (amount found, no date) | Auto-fill extracted fields; leave others blank with yellow "Review" badge |
| Non-receipt image (selfie, screenshot) | Low confidence (<0.3) → warning "This doesn't look like a receipt" |
| Duplicate receipt scanned | Hash-compare image; warn "Similar receipt detected on [date]. Save anyway?" |
| Very large image file (>10MB) | Client-side compression before upload; reject >20MB server-side |
| Multi-item receipt | Extract total amount only; itemized parsing deferred to future scope |

### F2 — Budget Limit Alerts

| Scenario | Handling |
|---|---|
| Budget set mid-month with existing spend over limit | Immediately trigger 100% alert on save |
| Category renamed/deleted that has a budget | Prompt user to re-assign budget or delete the limit |
| Multiple expenses cross threshold in rapid succession | Deduplicate alerts within a 5-minute window |
| User sets $0 budget | Reject: "Budget must be greater than $0" |

### F3 — Recurring Expense Auto-Entry

| Scenario | Handling |
|---|---|
| Billing day = 31 but month has 30 days | Roll to last day of month (e.g., Feb 28/29, Apr 30) |
| User deletes generated entry but rule exists | Rule persists independently; deleting a generated entry doesn't cancel the rule |
| App/server offline on scheduled date | Backfill missed entries on next startup with `source: "auto-backfill"` tag |
| Duplicate recurring rule for same category+amount | Warn: "A similar recurring expense already exists. Create anyway?" |
| 100+ recurring entries fire on 1st of month | Batch process; stagger notifications over 30-second window |

### F4 — MoM Comparison Chart

| Scenario | Handling |
|---|---|
| User has <2 months of data | Show single bar with message: "Add expenses next month to see trends" |
| Month with $0 spending | Render zero-height bar; tooltip shows "$0 — No expenses" |
| Category filter returns no results | Empty state: "No expenses found for selected categories" |
| Very large dataset (12+ months, 10+ categories) | Paginate to 12-month max viewport; virtual scroll for legend |

### F5 — Shared Expense Mode

| Scenario | Handling |
|---|---|
| Invite link used by unauthorized third party | Link expires after first join OR after 48 hours, whichever first |
| Both users log same expense (double entry) | Flag potential duplicates; prompt resolution |
| One user leaves shared space | Historical data becomes read-only for leaver |
| Fractional cent settlement (e.g., $100/3) | Round to 2 decimals; absorb ≤$0.01 variance silently |
| User tries to create >1 shared space | Block: "You can have 1 active shared space. Archive current to create new." |
| Shared expense edited by non-payer | Allow (both trusted); log audit trail |

---

## 6. KPIs & Acceptance Criteria

### Success Metrics

| KPI | Target | Measurement |
|---|---|---|
| Chart rendering time (F0b) | < 100ms on new expense add | Client-side profiling |
| CSV download time (F0c) | < 500ms from button click | Time to download trigger |
| OCR scan-to-save time (F1) | < 8 seconds (p95) | Image capture to saved expense |
| OCR field accuracy (F1) | ≥ 85% amount, ≥ 70% category | % scans requiring no manual correction |
| Budget alert latency (F2) | < 2 seconds | Time between save and alert render |
| Recurring entry reliability (F3) | 99.5% on-time | % entries created within ±1 hour of billing date |
| MoM chart render time (F4) | < 200ms for 12-month dataset | Client-side LCP |
| Shared space adoption (F5) | ≥ 15% of active users within 30 days | Analytics events |
| Settlement accuracy (F5) | 100% correctness | Automated test suite |

### Acceptance Criteria

**Core (F0)**
- [ ] GIVEN a freelancer on the dashboard, WHEN they add 5 expenses with different categories and amounts, THEN monthly totals and bar chart update immediately.
- [ ] GIVEN logged expenses, WHEN user clicks "Export to CSV", THEN a CSV file named `expenses_export.csv` with columns ID, Amount, Category, Date downloads.
- [ ] GIVEN an existing expense, WHEN user clicks delete, THEN the expense is removed and chart/totals update.

**OCR Receipt Scanner (F1)**
- [ ] GIVEN a clear receipt photo, WHEN scanned, THEN amount and date are auto-filled with ≥ 85% accuracy AND user can edit before saving.
- [ ] GIVEN a blurry/unreadable image, WHEN scanned, THEN a user-friendly error is shown AND form remains stable (no layout shift).

**Budget Limit Alerts (F2)**
- [ ] GIVEN a budget of $500 for "Marketing", WHEN a new expense pushes total to $400 (80%), THEN an amber alert banner is displayed within 2 seconds.
- [ ] GIVEN a hard budget limit of $500, WHEN spend reaches $500, THEN a red blocking dialog appears requiring explicit "Proceed Anyway" confirmation.

**Recurring Expense Auto-Entry (F3)**
- [ ] GIVEN a monthly recurring expense set for the 15th, WHEN the 15th arrives, THEN expense is auto-logged with `source: "auto"` AND notification sent.
- [ ] GIVEN a recurring rule with billing_day=31, WHEN month is February, THEN entry is created on Feb 28 (or 29 in leap year).

**MoM Comparison Chart (F4)**
- [ ] GIVEN 3+ months of data, WHEN user opens Analytics, THEN bar chart displays month-over-month totals with correct values.
- [ ] GIVEN category filter set to "Meals", WHEN applied, THEN chart updates to show only Meals data across all months.

**Shared Expense Mode (F5)**
- [ ] GIVEN User A creates a shared space, WHEN User B joins via invite code, THEN both see shared ledger AND personal ledgers remain separate.
- [ ] GIVEN 3 shared expenses ($60 by A, $40 by A, $50 by B), THEN settlement shows "B owes A $25.00".

---

## 7. Limitations & Risks

### Technical Limitations

| Limitation | Impact | Mitigation |
|---|---|---|
| Local storage data loss (core MVP) | Data lost if browser cache cleared | Persistent backend upgrade (required for F3, F5) |
| OCR accuracy on handwritten/faded receipts | Misread amounts need manual correction | Confidence scoring + mandatory review step |
| Recurring entry requires always-on backend | Missed entries if server down | Backfill mechanism on recovery; health monitoring |
| Shared mode limited to 2 users | No team/group splitting | Design N-user data model; enforce 2-user cap at app layer |
| MoM chart capped at 12 months | Long-term trends require scrolling | Sufficient for freelancer use case |
| Single currency assumption | Doesn't serve international freelancers | Store currency code field now; multi-currency in v3 |

### Security & Privacy Risks

| Risk | Mitigation |
|---|---|
| No auth in MVP → anyone on device can view/modify records | Introduce auth as prerequisite for advanced features |
| Receipt images may contain PII (name, card numbers) | Strip EXIF on upload; no raw image persistence beyond 24 hours |
| Shared space exposes financial data between users | Explicit consent flow on join; instant leave & revoke |
| Invite link interception | Time-expiring + single-use codes; HTTPS-only |

### Business Risks

| Risk | Mitigation |
|---|---|
| OCR API costs scale with usage | Daily scan cap (20/day free tier); cache vendor→category mappings |
| Feature bloat overwhelming "simple" value prop | Progressive disclosure: advanced features gated behind tabs/settings |
| Recurring entries create forgotten "ghost" data | Monthly digest notification summarizing auto-logged entries; one-tap cancel |
