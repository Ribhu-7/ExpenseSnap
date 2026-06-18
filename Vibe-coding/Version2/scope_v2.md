## 1. Goal & Problem Statement (V2)
* **The Problem:** The MVP solves basic manual-to-digital expense entry, but freelancers still face slow manual data entry for receipts, budget overruns without proactive alerts, tedious repetitive entry for fixed costs, lack of long-term MoM trend visibility, and friction splitting costs with a business partner.
* **The Solution:** A V2 extension that adds OCR receipt scanning, budget limit alerts, recurring expense auto-entry, month-over-month analytics, and a 2-user shared ledger system.

## 2. Tech Stack Additions (V2)
* **Backend Upgrades:** Mandatory Firebase/Supabase integration (for Auth, persistent database, and real-time pub/sub for shared ledgers). Local-storage-only is insufficient for V2.
* **Third-Party APIs:** Google Vision API or Tesseract.js (for OCR receipt processing).
* **Task Scheduling:** Server-side cron jobs or service workers (for automated recurring expense injection).
* **Charts:** Recharts extended with multi-dataset support for the MoM comparison.

## 3. Advanced Features & Acceptance Criteria (F1-F5)
| Feature number | Feature name | Description | Acceptance Criteria |
| --- | --- | --- | --- |
| F1 | OCR Receipt Scanner | Camera/upload receipt scanner with automatic extraction and form autofill | AC-01: Upload receipt, parse amount & date with >=85% accuracy.<br>AC-02: Suggest category based on vendor mapping with >=70% accuracy.<br>AC-03: UI stays stable; zero vertical layout shifts or keyboard jumps during loading.<br>AC-04: User can review and edit all parsed fields before saving. |
| F2 | Budget Limit Alerts | Monthly soft (80%) and hard (100%) spending caps per category | AC-05: Display amber banner when spend hits >=80% of limit.<br>AC-06: Display red blocking modal at >=100% requiring explicit "Proceed Anyway" confirmation.<br>AC-07: Budget limits set mid-month trigger alerts immediately if current spend exceeds limit. |
| F3 | Recurring Expense Auto-Entry | Automated daily/weekly/monthly expense entries on scheduled billing dates | AC-08: Logs entries on billing date with `source: "auto"` without requiring app launch.<br>AC-09: Rollover billing day = 31 to last day of shorter months (e.g., Feb 28/29).<br>AC-10: Backfill missed entries on next startup/sync if system was offline. |
| F4 | MoM Comparison Chart | Analytics view comparing current month spend against previous months | AC-11: Bar/line comparison chart filters by category and range (3/6/12 months).<br>AC-12: Renders $0-spend months as zero-height bars with clear tooltip data.<br>AC-13: Show placeholder state if less than 2 months of data exist. |
| F5 | Shared Expense Mode | 2-user shared space with 50/50 split tracking and settlement calculation | AC-14: User A generates invite code; User B enters code to join shared ledger.<br>AC-15: Show combined ledger and live "Who owes whom" 50/50 balance.<br>AC-16: Private personal ledgers and shared spaces remain completely isolated via workspace IDs. |

## 4. UI/UX Standards (V2 Impacts)
* **Layout Stability (OCR):** Overlay loading skeletons must prevent layout shift (CLS = 0) during OCR uploads.
* **Alerts & Dialogs:** Smooth slide-in toast notifications for soft budget warnings (80%) and centered blocking confirmation modals for hard budget limits (100%).

## 5. Out of Scope (V2)
* Team/group expense splitting beyond 2 users.
* Multi-currency support (single currency assumed for all F1-F5 logic).
* AI-powered auto-categorization beyond static vendor→category mapping.
* Full itemized receipt parsing (only total amount is extracted for F1).
