# Developer Handover & Workflow Document: ExpenseSnap

## 1. Project Overview

* **Project Name**: ExpenseSnap
* **Business Purpose**: Provide freelancers, solopreneurs, and 2-person micro-teams with a lightweight, secure web application to log, track, automate, and split business expenses without the friction of complex setups or enterprise accounting software.
* **Problem Statement**: Freelancers track business expenses manually using physical notebooks — a process that is error-prone, lacks visual insights, offers no spending limits, and makes tax exporting tedious. As businesses grow, repetitive monthly expenses, splitting shared costs with co-founders, and paper receipt digitization become significant friction points.
* **Solution Summary**: A web application supporting instant manual/OCR expense logging, real-time monthly visualizations, CSV exports with formula sanitization, category budget caps, scheduled recurring costs, month-over-month analytics, and collaborative 2-user split tracking.
* **Key Features**:
  - **Core Expense CRUD:** Manual entry and deletion with input validation.
  - **OCR Receipt Scanner:** Simulated text parsing (amount, vendor, date) and form autocomplete.
  - **Category Budgets:** 80% soft alert warnings and 100% hard blocking confirmations.
  - **Recurring Expenses:** Automated daily/weekly/monthly cost generation with month-end rollover and backfills.
  - **MoM Analytics:** Interactive Bar/Line chart comparisons with category and range filters.
  - **Shared Split Ledgers:** 50/50 split tracking with remote invite codes and live settlement logs.
  - **CSV Export:** One-click downloads with formula injection sanitization.

---

## 2. Technology Stack

| Layer | Technologies |
|---|---|
| **Frontend** | React 19, TypeScript, Vite, Tailwind CSS v3, Formik, Yup, Recharts, Lucide React, date-fns, uuid |
| **Backend & Sync** | Context API Client Simulation Engine (simulating magic-link auth, real-time spaces, and scheduler cron tasks) |
| **Database & Caching** | LocalStorage (`expense_snap_data`, `expense_snap_budgets`, `expense_snap_recurring_rules`, `expense_snap_shared_space`) |
| **Testing** | React Testing Library, Jest, Playwright (configured for type-safe execution) |

---

## 3. Architecture Overview

* **High-Level Architecture:** Single Page Application (SPA) utilizing a **Mock Services Engine** built on React Context and Hooks. Data is stored in partitioned LocalStorage keys to maintain workspace isolation and survive page refreshes.
* **Request/Data Flow:**
  - **UI Interaction** $\rightarrow$ **Validation Formik/Yup** $\rightarrow$ **Budget Interceptors** $\rightarrow$ **Auth/Expense Contexts** $\rightarrow$ **Storage Service** $\rightarrow$ **Partitioned LocalStorage**.
* **Major Services:**
  - `AuthProvider`: Manages active user sessions (Alex/Taylor), invite code hooks, and workspace IDs.
  - `useExpenses`: Fetches and filters expense arrays by workspace ID, handles manual entries and background scheduler injections.
  - `useBudgets`: Tracks spending limits, calculates category utilization, and triggers warning levels (`ok | soft | hard`).
  - `useRecurring`: Manages templates and triggers chronological auto-generation of due entries.

---

## 4. Project Structure

```text
src/
├── charts/             # Recharts visualizations (MonthlySummaryChart)
├── components/         # Reusable feature-specific UI components
│   ├── AnalyticsTab.tsx              # MoM comparison charts and category filters
│   ├── BudgetAlerts.tsx              # Budget settings forms, banners, and blocking modals
│   ├── ExpenseForm.tsx               # Formik expense capture with OCR & scheduler hooks
│   ├── ExpenseList.tsx               # Transaction list showing payment/source markers
│   ├── OCRScanner.tsx                # Camera drop zones and skeleton loaders (CLS = 0)
│   ├── RecurringExpensesManager.tsx  # Automated cycle settings
│   └── SharedSpaceManager.tsx        # Invitation codes and 50/50 balance calculators
├── context/            # Auth and multi-workspace providers (AuthContext.tsx)
├── features/           # Feature pages (expenses/pages/Dashboard.tsx)
├── hooks/              # Custom React hooks (useExpenses, useBudgets, useRecurring)
├── storage/            # Data storage logic and LocalStorage partitions (localStorage.ts)
├── types/              # Type definitions (index.ts)
├── utils/              # Helper utilities (csvExport.ts, ocrParser.ts)
└── main.tsx            # App bootstrapping
```

---

## 5. Module Summary

| Module | Purpose | Key Components | Dependencies |
|---|---|---|---|
| **Auth & Workspace** | Manages logged-in personas and isolates personal/shared ledgers | `AuthContext.tsx` | React Context |
| **Expense CRUD** | Captures manual or scanner inputs and handles deletion | `ExpenseForm.tsx`, `ExpenseList.tsx` | Formik, Yup, uuid |
| **OCR Scan** | Simulates receipt parsing, showing loader cards without layout shift | `OCRScanner.tsx`, `ocrParser.ts` | Lucide React |
| **Budgets** | Intercepts submissions to enforce spending thresholds | `BudgetAlerts.tsx`, `useBudgets.ts` | React Context |
| **Scheduler** | Auto-logs recurring templates on billing dates | `RecurringExpensesManager.tsx`, `useRecurring.ts` | date-fns |
| **Analytics** | Plots month-over-month charts comparing expenditures | `AnalyticsTab.tsx` | Recharts, date-fns |
| **Shared Spaces** | Connects two users to compute splits and settle balances | `SharedSpaceManager.tsx` | React Context |

---

## 6. Database Overview

All data is partitioned inside browser LocalStorage:

- `expense_snap_data`: Array of `Expense` objects:
  ```typescript
  { id, amount, category, date, vendor?, paidBy?, workspaceId, source? }
  ```
- `expense_snap_budgets`: Array of `Budget` objects:
  ```typescript
  { id, category, limitAmount, limitType, month }
  ```
- `expense_snap_recurring_rules`: Array of `RecurringRule` objects:
  ```typescript
  { id, amount, category, frequency, billingDay, status, workspaceId, lastRunDate? }
  ```
- `expense_snap_shared_space`: Active shared space metadata:
  ```typescript
  { id, inviteCode, name, members }
  ```

---

## 7. API Overview

Features use simulated client-side endpoints mimicking real REST APIs:

- `POST /api/v2/receipts/scan` $\rightarrow$ Handled by `parseReceiptSimulated(fileName, fileSize)`.
- `GET /api/v2/budgets?month=YYYY-MM` $\rightarrow$ Handled by `useBudgets.getBudgetStatus()`.
- `POST /api/v2/recurring` $\rightarrow$ Handled by `useRecurring.addRule()`.
- `GET /api/v2/analytics/mom` $\rightarrow$ Handled by `AnalyticsTab` interval filter compilation.
- `GET /api/v2/shared-spaces/settlement` $\rightarrow$ Handled by `SharedSpaceManager` 50/50 balance calculators.

---

## 8. Environment & Setup

* **Required Environment Variables:** None for local development.
* **Installation:** Run `npm install` inside the `Vibe-coding/expense_snap` folder.
* **Local Run:** Run `npm run dev` to start the Vite dev server.
* **Build Compilation:** Run `npm run build` to verify type safety and bundle files.

---

## 9. Business Workflows

### Workflow 1: Scanning a Receipt via OCR
1. User clicks **"Scan Paper Receipt"** in the sidebar.
2. Selects an image file $\rightarrow$ a skeleton card overlay appears (maintaining form dimensions, CLS = 0).
3. The mock OCR parser extracts parameters:
   - Starbucks $\rightarrow$ Meals, AWS $\rightarrow$ Software, Uber $\rightarrow$ Travel.
4. Auto-fills the main form. User reviews, edits any fields, and clicks **"Add to Ledger"**.

### Workflow 2: Enforcing Budgets & Alerts
1. User configures a budget limit in **"Budgets & Recurring"** (e.g., $100 for Software).
2. When submitting an expense:
   - Spend hits 80% ($80) $\rightarrow$ An amber alert toast displays at the top of the dashboard.
   - Spend hits 100% ($100) $\rightarrow$ A red blocking modal interrupts the save. User must click **"Proceed Anyway"** to override.

### Workflow 3: Background Recurring Entry Auto-Generation
1. User saves an expense with **"Schedule as Recurring Cost"** checked.
2. The hook `useRecurring` runs on load/sync.
3. For all rules:
   - Daily: Injects every day.
   - Weekly: Injects on weekday indexes.
   - Monthly: Injects on day-of-month.
4. Rollover: If set to day 31, auto-runs on Feb 28/29 or Apr 30 in shorter months.
5. Backfill: If the app was offline, parses missed dates and auto-logs them with `source: "auto-backfill"`.

### Workflow 4: Shared Space Splits & Settlement
1. Alex logs in $\rightarrow$ creates a Shared Space $\rightarrow$ shares the generated 6-character code.
2. Taylor logs in $\rightarrow$ enters the code to connect spaces.
3. Both switch their workspace to **"Shared Space"**.
4. Adding an expense tags it with the active payer (`paidBy: "user_a" | "user_b"`).
5. The settlement engine computes splits:
   - Total Shared Spend = $500 (Alex paid $350, Taylor paid $150).
   - average = $250. Alex is owed $100.
   - UI displays: **"Taylor owes Alex $100.00"**.

---

## 10. Security & Error Handling

* **Authentication & Workspace Isolation:** Ledger arrays are strictly filtered by `workspaceId`. Active workspace controls prevent cross-contamination.
* **XSS & Input Sanitization:** Yup schemas block special tags (`<>&"'`) in category fields.
* **CSV Injection Protection:** The CSV export utility automatically prepends a single quote (`'`) to any string starting with `=`, `+`, `-`, or `@`.
* **OCR Failures:** If a desktop screenshot or blurry file is uploaded, the parser rejects it and displays a warning toast. The form remains stable.
* **Verbatim Module Syntax:** Imports of interfaces use explicit `import type` syntax to satisfy strict compiler requirements.

---

## 11. Testing Overview

| Test Type | Coverage | Tools |
|---|---|---|
| **Unit Tests** | Storage partition validators, split formulas | Jest |
| **Component Tests** | OCR loaders, budget warnings, blocking modal overrides | React Testing Library |
| **E2E Integration** | Multi-workspace toggles, user switching splits | Playwright |

---

## 12. Troubleshooting Guide

* **Vite fails on build:** Ensure all type imports in `.tsx` files use `import type` to comply with the Verbatim Module Syntax rules.
* **Recurring expenses not creating:** Ensure your browser is not blocking LocalStorage writes. The scheduler checks rules only on startup or workspace changes.
* **MoM Chart not rendering:** Analytics require data across at least 2 distinct calendar months before displaying. If there is less data, it shows the pending info state.
* **Layout Shifts:** Check that skeleton cards have fixed heights (e.g. `min-h-[120px]`) matching the standard trigger boxes to maintain CLS = 0.
