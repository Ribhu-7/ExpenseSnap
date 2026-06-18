# PRODUCT REQUIREMENTS DOCUMENT (PRD) — V2: Advanced Features

> **Extends:** [prd.md (MVP)](file:///Users/neosoft/Documents/ExpenseSnap/Vibe-Coding/Vibe-coding/Agent/prd.md)
> **Version:** 2.0 | **Date:** 2026-06-12

---

## 1. Problem Statement

The MVP solves basic manual-to-digital expense entry. However, freelancers still face:

| Pain Point | Feature Solution |
|---|---|
| Manual amount typing is slow & error-prone for paper receipts | **OCR Receipt Scanner** |
| No spending guardrails → budget overruns discovered too late | **Budget Limit Alerts** |
| Repetitive entry of fixed costs (rent, SaaS tools) every month | **Recurring Expense Auto-Entry** |
| No trend visibility across months; decisions are gut-based | **MoM Comparison Chart** |
| Co-founders / partners splitting shared costs resort to spreadsheets | **Shared Expense Mode** |

**Target Users:** Freelancers, solopreneurs, and 2-person micro-teams who have outgrown notebook tracking and need lightweight financial intelligence without full accounting software.

---

## 2. Solution Overview

### Feature Matrix

| # | Feature | Priority | Dependency |
|---|---|---|---|
| F1 | OCR Receipt Scanner | P0 | Cloud OCR API (Google Vision / Tesseract.js) |
| F2 | Budget Limit Alerts | P0 | Expense CRUD (MVP) |
| F3 | Recurring Expense Auto-Entry | P1 | Scheduler (cron / service worker) |
| F4 | MoM Comparison Chart | P1 | Expense data ≥ 2 months |
| F5 | Shared Expense Mode | P2 | Auth system, real-time DB (Firebase/Supabase) |

### Architecture Impact
- **Auth Required:** F5 mandates user authentication; recommended to introduce lightweight auth (email magic-link or OAuth) as a platform prerequisite for F2–F5.
- **Backend Required:** F3 (server-side scheduler) and F5 (shared ledger) require a persistent backend. Local-storage-only mode from MVP must be upgraded.
- **New Dependencies:** OCR API client, charting library upgrade (multi-dataset support), real-time pub/sub for F5.

---

## 3. User Flow

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
- Confidence indicator per field (e.g., green check = high confidence, yellow = review suggested).

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

Response 422 (Unprocessable):
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

## 5. Edge Cases

### F1 — OCR Receipt Scanner

| Scenario | Handling |
|---|---|
| Blurry / unreadable image | Return `OCR_PARSE_FAILED` error; show "Could not read receipt. Try again with better lighting." |
| Partial parse (amount found, no date) | Auto-fill extracted fields, leave others blank with yellow "Review" badge |
| Non-receipt image (selfie, screenshot) | OCR returns low confidence (<0.3) → show warning "This doesn't look like a receipt" |
| Duplicate receipt scanned | Hash-compare image; warn "Similar receipt detected on [date]. Save anyway?" |
| Very large image file (>10MB) | Client-side compression before upload; reject >20MB server-side with clear error |
| Multi-item receipt | Extract total amount only (v2 scope); itemized parsing deferred to v3 |

### F2 — Budget Limit Alerts

| Scenario | Handling |
|---|---|
| Budget set mid-month with existing spend already over limit | Immediately trigger 100% alert on save; don't wait for next expense |
| Category renamed/deleted that has a budget | Cascade: prompt user to re-assign budget to new category or delete the limit |
| Multiple small expenses cross threshold in rapid succession | Deduplicate alerts within a 5-minute window; show single consolidated notification |
| User sets $0 budget | Reject with validation error: "Budget must be greater than $0" |
| Currency mismatch (future consideration) | v2 assumes single currency; flag for v3 multi-currency support |

### F3 — Recurring Expense Auto-Entry

| Scenario | Handling |
|---|---|
| Billing day = 31 but month has 30 days | Roll to last day of month (e.g., Feb 28/29, Apr 30) |
| User deletes the original expense but recurring rule exists | Rule persists independently; deleting a generated entry doesn't cancel the rule |
| App/server offline on scheduled date | Backfill missed entries on next startup/sync with `source: "auto-backfill"` tag |
| User creates duplicate recurring rule for same category+amount | Warn: "A similar recurring expense already exists. Create anyway?" |
| 100+ recurring entries fire on the 1st of the month | Batch process; stagger notifications over 30-second window to avoid spam |

### F4 — MoM Comparison Chart

| Scenario | Handling |
|---|---|
| User has <2 months of data | Show single bar with message: "Add expenses next month to see trends" |
| Month with $0 spending | Render as zero-height bar (not hidden); tooltip shows "$0 — No expenses" |
| Category filter returns no results | Empty chart state: "No expenses found for selected categories" |
| Very large dataset (12+ months, 10+ categories) | Paginate to 12-month max viewport; virtual scroll for legend |

### F5 — Shared Expense Mode

| Scenario | Handling |
|---|---|
| Invite link used by a third unauthorized user | Link expires after first successful join OR after 48 hours, whichever is first |
| Both users log the same expense (double entry) | Flag potential duplicates (same amount + date + category within shared space); prompt resolution |
| One user leaves shared space | Historical data becomes read-only for leaver; remaining user retains full access |
| Settlement shows fractional cents (e.g., $33.33 owed on $100/3) | Round to 2 decimal places; absorb ≤$0.01 variance silently |
| User tries to create >1 shared space (v2 limit) | Block with: "You can have 1 active shared space. Archive current space to create a new one." |
| Shared expense is edited by non-payer | Allow edits (both are trusted); log audit trail of who modified what |

---

## 6. KPIs & Acceptance Criteria

### Success Metrics

| KPI | Target | Measurement |
|---|---|---|
| OCR scan-to-save time | < 8 seconds (p95) | Time from image capture to saved expense |
| OCR field accuracy | ≥ 85% for amount, ≥ 70% for category suggestion | % of scans requiring no manual correction |
| Budget alert delivery latency | < 2 seconds after threshold-crossing expense save | Time delta between save and alert render |
| Recurring entry reliability | 99.5% on-time execution | % of scheduled entries created within ±1 hour of billing date |
| MoM chart render time | < 200ms for 12-month dataset | Client-side profiling (LCP) |
| Shared space adoption | ≥ 15% of active users create a shared space within 30 days | Analytics event tracking |
| Settlement accuracy | 100% mathematical correctness | Automated test suite |

### Acceptance Criteria

- [ ] **F1:** GIVEN a clear receipt photo, WHEN scanned, THEN amount and date fields are auto-filled with ≥ 85% accuracy AND the user can edit all fields before saving.
- [ ] **F1:** GIVEN a blurry/unreadable image, WHEN scanned, THEN a user-friendly error is shown AND the form remains stable (no layout shift).
- [ ] **F2:** GIVEN a budget of $500 for "Marketing", WHEN a new expense pushes total to $400 (80%), THEN an amber alert banner is displayed within 2 seconds.
- [ ] **F2:** GIVEN a hard budget limit of $500, WHEN spend reaches $500, THEN a red blocking dialog appears requiring explicit "Proceed Anyway" confirmation.
- [ ] **F3:** GIVEN a monthly recurring expense set for the 15th, WHEN the 15th arrives, THEN the expense is auto-logged with `source: "auto"` AND a notification is sent.
- [ ] **F3:** GIVEN a recurring rule with billing_day=31, WHEN the month is February, THEN the entry is created on Feb 28 (or 29 in leap year).
- [ ] **F4:** GIVEN 3+ months of expense data, WHEN the user opens Analytics, THEN a bar chart displays month-over-month totals with correct values.
- [ ] **F4:** GIVEN category filter set to "Meals", WHEN applied, THEN the chart updates to show only Meals data across all displayed months.
- [ ] **F5:** GIVEN User A creates a shared space, WHEN User B joins via invite code, THEN both see a shared ledger AND individual personal ledgers remain separate.
- [ ] **F5:** GIVEN 3 shared expenses ($60 by A, $40 by A, $50 by B), THEN settlement shows: "B owes A $25.00".

---

## 7. Limitations & Risks

### Technical Limitations

| Limitation | Impact | Mitigation |
|---|---|---|
| OCR accuracy on handwritten / faded receipts | Misread amounts require manual correction | Confidence scoring + mandatory review step |
| Recurring entry requires always-on backend | Missed entries if server is down | Backfill mechanism on recovery; service health monitoring |
| Shared mode limited to 2 users (v2) | Cannot support team/group expense splitting | Design data model for N-user from day 1; enforce 2-user cap at application layer |
| MoM chart limited to 12 months viewport | Long-term trend analysis requires scrolling | Sufficient for freelancer use case; revisit if enterprise pivot |
| Single currency assumption across all features | Doesn't serve international freelancers with multi-currency income | Flag for v3; store currency code field now for forward compatibility |

### Security & Privacy Risks

| Risk | Mitigation |
|---|---|
| Receipt images may contain PII (name, card numbers) | Strip EXIF data on upload; process OCR server-side; do not persist raw images beyond 24 hours |
| Shared space exposes financial data between users | Explicit consent flow on join; ability to leave and revoke access instantly |
| Invite link interception | Time-expiring + single-use invite codes; HTTPS-only |

### Business Risks

| Risk | Mitigation |
|---|---|
| OCR API costs scale with usage | Implement daily scan cap (e.g., 20/day free tier); cache vendor→category mappings to reduce re-processing |
| Feature bloat overwhelming the "simple" value proposition | Progressive disclosure: advanced features gated behind tabs/settings, not cluttering the core expense form |
| Recurring entries create "ghost" data users forget about | Monthly digest email/notification summarizing auto-logged entries; easy one-tap cancel |
