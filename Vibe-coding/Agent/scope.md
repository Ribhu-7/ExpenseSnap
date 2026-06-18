## 1. Goal & Problem Statement
* **The Problem:** Freelancers track business expenses manually using physical notebooks — a process that is error-prone, lacks visual insights, offers no spending guardrails, and makes tax reporting tedious. As their business grows, they face repetitive data entry for recurring costs, lack of month-over-month trend visibility, difficulty digitizing paper receipts quickly, and split tracking disputes with partners.
* **The Solution:** A lightweight web application that enables instant expense logging (manually or via OCR scan), category budget alerts, scheduled recurring entries, month-over-month analytics, 2-user shared ledger split tracking, and CSV export.

## 2. Tech Stack
* **Frontend:** React + TypeScript, Tailwind CSS, Recharts (for dynamic single & multi-dataset charts)
* **Backend & Database:** Node.js, Express, and PostgreSQL (for authentication, real-time shared ledgers, and persistent database storage from day one)
* **Auth:** JWT-based User Authentication (Login/Signup via PostgreSQL, required for all features)
* **Third-Party APIs:** Google Vision API or Tesseract.js (for receipt OCR processing)
* **Task Scheduling:** Server-side cron/service worker (for automated recurring expense injection)

## 3. Core & Advanced Features & Acceptance Criteria
| Feature number | Feature name | Description | Acceptance Criteria |
| --- | --- | --- | --- |
| 1 | User Auth & Expense Logging | Secure login via PostgreSQL. Add, view, edit, and delete expenses. | AC-01: User can login securely.<br>AC-02: User can add expense with valid amount/category/date.<br>AC-03: User can edit existing expense.<br>AC-04: User can delete expense.<br>AC-05: Invalid inputs are blocked. |
| 2 | Monthly Summary & Chart | Real-time monthly summary grouped by category with dynamic bar chart visualization | AC-05: Bar chart renders within 100ms of new expense.<br>AC-06: Monthly totals update immediately.<br>AC-07: Empty state shown when no expenses exist. |
| 3 | CSV Export | One-click download of all expense records in standard CSV format | AC-08: CSV downloads within 500ms.<br>AC-09: File contains ID, Amount, Category, Date columns.<br>AC-10: All expenses included in export. |
| 4 | OCR Receipt Scanner | Camera/upload receipt scanner with automatic extraction and form autofill | AC-11: Upload receipt, parse amount & date with >=85% accuracy.<br>AC-12: Suggest category based on vendor mapping with >=70% accuracy.<br>AC-13: UI stays stable; zero vertical layout shifts or keyboard jumps during loading.<br>AC-14: User can review and edit all parsed fields before saving. |
| 5 | Budget Limit Alerts | Monthly soft (80%) and hard (100%) spending caps per category | AC-15: Display amber banner when spend hits >=80% of limit.<br>AC-16: Display red blocking modal at >=100% requiring explicit "Proceed Anyway" confirmation.<br>AC-17: Budget limits set mid-month trigger alerts immediately if current spend exceeds limit. |
| 6 | Recurring Expense Auto-Entry | Automated daily/weekly/monthly expense entries on scheduled billing dates | AC-18: Logs entries on billing date with `source: "auto"` without requiring app launch.<br>AC-19: Rollover billing day = 31 to last day of shorter months (e.g., Feb 28/29).<br>AC-20: Backfill missed entries on next startup/sync if system was offline. |
| 7 | MoM Comparison Chart | Analytics view comparing current month spend against previous months | AC-21: Bar/line comparison chart filters by category and range (3/6/12 months).<br>AC-22: Renders $0-spend months as zero-height bars with clear tooltip data.<br>AC-23: Show placeholder state if less than 2 months of data exist. |
| 8 | Shared Expense Mode | 2-user shared space with 50/50 split tracking and settlement calculation | AC-24: User A generates invite code; User B enters code to join shared ledger.<br>AC-25: Show combined ledger and live "Who owes whom" 50/50 balance.<br>AC-26: Private personal ledgers and shared spaces remain completely isolated. |

## 4. UI/UX Standards
* **Theme & Style:** Clean, minimalist design with soft shadows, dynamic glassmorphism panels, and a cohesive dark/light palette
* **Layout:** Mobile-first responsive grids with overlay loading skeletons to prevent layout shift during OCR uploads or chart rendering
* **Alerts & Dialogs:** Smooth slide-in toast notifications for budget warnings (80%) and centered blocking confirmation modals for hard budget limits (100%)

## 5. Out of Scope
* Integration with external bank account feeds or card transaction APIs
* Multi-currency support (single currency assumed; currency code field stored for future expansion)
* Multi-user shared spaces beyond 2 users (no team/group splits)
* AI-based category matching beyond static vendor-to-category mapping lists

