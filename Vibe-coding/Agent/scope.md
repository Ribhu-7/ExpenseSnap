## 1. Goal & Problem Statement
* **The Problem:** Freelancers track business expenses manually using physical notebooks — a process that is error-prone, lacks visual insights, offers no spending guardrails, and makes tax reporting tedious. As their business grows, they face repetitive data entry for recurring costs, lack of month-over-month trend visibility, difficulty digitizing paper receipts quickly, and split tracking disputes with partners.
* **The Solution:** A lightweight web application that enables instant expense logging (manually or via OCR scan), category budget alerts, scheduled recurring entries, month-over-month analytics, 2-user shared ledger split tracking, and CSV export.

## 2. Tech Stack
* **Frontend:** React + TypeScript, Tailwind CSS, Recharts (for dynamic single & multi-dataset charts)
* **Backend & Database:** Firebase or Supabase (for authentication, real-time shared ledgers, and database persistence to replace local-only storage)
* **Auth:** Magic-link email login or OAuth (prerequisite for shared space and sync features)
* **Third-Party APIs:** Google Vision API or Tesseract.js (for receipt OCR processing)
* **Task Scheduling:** Server-side cron/service worker (for automated recurring expense injection)

## 3. Core & Advanced Features & Acceptance Criteria
| Feature number | Feature name | Description | Acceptance Criteria |
| --- | --- | --- | --- |
| 1 | Expense Logging (CRUD) | Add, view, edit, and delete expenses with amount, category, and date fields | AC-01: User can add expense with valid amount/category/date.<br>AC-02: User can edit existing expense.<br>AC-03: User can delete expense.<br>AC-04: Invalid inputs are blocked with visual feedback. |
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

# KPI Matrix
| KPI Number | KPI Name | Description | Criteria |
| --- | --- | --- | --- |
| KPI-CRUD-01 | Input Validation Latency | UI response to invalid input submission | Border highlight and warning message display in < 150ms |
| KPI-CRUD-02 | In-Memory CRUD Latency | Speed of updating database records | Database write and state change triggers occur in < 50ms |
| KPI-CRUD-03 | Form Fields Auto-Reset | Input fields clear after submission | Form resets to default empty inputs within 100ms of add |
| KPI-CHART-01 | Chart Rendering Latency | Speed of re-rendering SVG/Canvas chart | Chart visual updates and renders new data in < 100ms |
| KPI-CHART-02 | Aggregation Precision | Category summation math correctness | Categorized totals show zero rounding errors (100% accuracy) |
| KPI-CHART-03 | Chart Empty State Display | Check empty chart placeholder UX | Zero-expense state renders placeholder text and empty chart graphics |
| KPI-EXP-01 | Export Completeness | CSV record count validation | CSV row count matches database records |
| KPI-EXP-02 | CSV Export Speed | Download delay from button click | CSV download trigger begins in < 300ms |
| KPI-EXP-03 | CSV Formula Sanitization | Security protection against CSV injection | Values starting with `=`, `+`, `-`, or `@` prepended with `'` |
| KPI-OCR-01 | Scan-to-Save Latency | OCR capture to save duration | < 8 seconds at p95 |
| KPI-OCR-02 | Amount Extraction Accuracy | Scan amount parsing success rate | >= 85% accuracy without manual corrections |
| KPI-OCR-03 | Category Suggestion Accuracy | OCR auto-category correctness | >= 70% accuracy on suggested categories |
| KPI-OCR-04 | Layout Stability on Processing | UI layout shifting during OCR loading | Cumulative Layout Shift (CLS) = 0 during OCR loading |
| KPI-BGT-01 | Alert Delivery Latency | Budget warning display trigger speed | Alert banner renders in < 2 seconds of crossing threshold |
| KPI-BGT-02 | Threshold Calculation Accuracy | Spend vs. budget mathematical check | 100% mathematical accuracy; zero false alerts |
| KPI-BGT-03 | 100% Hard Alert Blocking | Blocking confirmation on budget breach | Renders blocking confirmation; halts save until override |
| KPI-REC-01 | On-Time Execution Rate | Scheduled recurring entry execution | >= 99.5% of entries created within ±1 hour of billing date |
| KPI-REC-02 | Backfill Reliability | Auto-backfill for system downtime | 100% of missed entries backfilled with `source: "auto-backfill"` |
| KPI-MOM-01 | MoM Chart Render Time | Analytics viewport rendering performance | < 200ms render for 12-month dataset |
| KPI-MOM-02 | Filter Responsiveness | Chart update on category filter change | < 300ms chart animation transitions |
| KPI-SHR-01 | Invite Link Generation | Shareable code creation delay | < 1 second |
| KPI-SHR-02 | Join Flow Completion | Invited user join process speed | < 30 seconds from link click to shared ledger view |
| KPI-SHR-03 | Settlement Accuracy | Split tracking math validation | 100% balance accuracy ("Who owes whom") |
| KPI-SHR-04 | Ledger Isolation | Personal ledger data privacy | 100% isolation; personal entries never leak into shared ledger |

# Development Timeline
| Sprint | Focus Area | Deliverables |
| --- | --- | --- |
| Sprint 1 | Core Setup & CRUD | Initial React/TypeScript setup, Expense entry UI forms, database integration, input validation, and list view. |
| Sprint 2 | Charting & Summary | Integration of charting library, monthly aggregation logic, dynamic chart updates, and responsiveness. |
| Sprint 3 | Export & Polish | CSV download button implementation, CSV injection sanitization, UI/UX polish, empty states, and core QA. |
| Sprint 4 | OCR Receipt Scanner | Camera/file upload UI, OCR API integration, auto-fill form logic, layout stability enforcement, confidence UI. |
| Sprint 5 | Budget Limit Alerts | Budget settings UI, category cap tracking engine, 80% soft alert banner, 100% hard blocking dialog, alert deduplication. |
| Sprint 6 | Recurring Expenses | Recurring expense schedule options, server-side cron scheduler, backfill service on startup, entry notifications. |
| Sprint 7 | MoM Analytics Chart | Analytics view, multi-dataset bar/line chart toggle, category/range filters, $0-spend month handlers. |
| Sprint 8 | Shared Expense Mode | Auth setup, invite code creation/join flows, combined shared ledger, 50/50 settlement calculator, ledger privacy controls. |
| Sprint 9 | Integration & QA | End-to-end user testing, load and performance profiling, security review (XSS/CSV injection/auth logic), final UAT. |

# Success Criteria
| Category | Success Metric | Target |
| --- | --- | --- |
| Operational | Core CRUD Capability | 100% successful CRUD actions verified in UAT |
| Technical | Chart Sync Responsiveness | Real-time chart updates within 100ms |
| Compliance | CSV Validation & Sanitization | CSV opens cleanly in Excel/Sheets with zero injection vulnerabilities |
| Usability | Layout Stability (CLS) | CLS = 0 during CRUD updates, OCR processing, and chart transitions |
| Performance | OCR Process Speed | Scan-to-save time < 8s (p95) |
| Performance | Page Load Speed | Initial page load LCP < 1.0s |
| Reliability | Recurring Entry Success | >= 99.5% on-time execution of automated entries |
| Security | Shared Ledger Isolation | 100% personal ledger privacy; unauthorized users cannot access shared spaces |
| Accuracy | Settlement Calculation | 100% correct balance computations for 50/50 splits |