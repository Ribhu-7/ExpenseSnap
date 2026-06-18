# KPI Document
 
For EACH identified module:
 
#### Module 1: Auth & Expense CRUD Module (Operational, Data Integrity & Security)
 
| KPI Number | KPI Name | Description | Criteria |
| --- | --- | --- | --- |
| KPI-AUTH-01 | Login/Register Latency | Measures API response time for authentication requests. | Auth API responses in < 200ms. |
| KPI-CRUD-01 | Input Validation Latency | Measures UI responsiveness to invalid input submission. | Border highlight and warning message display in < 150ms. |
| KPI-CRUD-02 | Database CRUD Latency | Tracks speed of saving/deleting records in PostgreSQL database. | Database updates and state change triggers occur in < 50ms. |
| KPI-CRUD-03 | Form Fields Auto-Reset | Checks if the input form resets after submission. | Form resets to default empty inputs within 100ms of add event. |
 
#### Module 2: Monthly Summary & Chart Module (Technical & UI/UX)
 
| KPI Number | KPI Name | Description | Criteria |
| --- | --- | --- | --- |
| KPI-CHART-01 | Chart Rendering Latency | Measures speed of re-rendering the SVG/Canvas bar chart. | Chart visual re-renders and displays new heights in < 100ms. |
| KPI-CHART-02 | Aggregation Precision | Validates category summation logic. | Categorized totals show zero rounding errors (100% accuracy). |
| KPI-CHART-03 | Chart Empty State Display | Checks if empty chart shows default message. | Zero-expense state renders placeholder text and empty chart graphics. |
 
#### Module 3: CSV Export Module (Security & Compliance)
 
| KPI Number | KPI Name | Description | Criteria |
| --- | --- | --- | --- |
| KPI-EXP-01 | Export Completeness | Ensures all records are captured in the CSV. | Number of rows in CSV matches the number of logged expenses. |
| KPI-EXP-02 | CSV Export Speed | Measures latency from button click to download start. | CSV download trigger begins in < 300ms. |
| KPI-EXP-03 | CSV Formula Sanitization | Protects against CSV injection attacks. | All values beginning with `=`, `+`, `-`, or `@` are prepended with a single quote (`'`). |

#### Module 4: OCR Receipt Scanner Module (Accuracy & Performance)

| KPI Number | KPI Name | Description | Criteria |
| --- | --- | --- | --- |
| KPI-OCR-01 | Scan-to-Save Latency | End-to-end time from image capture/upload to saved expense. | < 8 seconds at p95. |
| KPI-OCR-02 | Amount Extraction Accuracy | % of scans where extracted amount requires no manual correction. | ≥ 85% accuracy. |
| KPI-OCR-03 | Category Suggestion Accuracy | % of scans where suggested category is accepted without change. | ≥ 70% accuracy. |
| KPI-OCR-04 | Layout Stability on Processing | Zero cumulative layout shift during OCR processing overlay. | CLS = 0 during scan→parse→autofill cycle. |
| KPI-OCR-05 | Error Handling UX | Graceful failure on unreadable/non-receipt images. | User-friendly error message displayed in < 2 seconds; form remains stable. |
| KPI-OCR-06 | Image Size Handling | Client-side compression for large uploads. | Images > 10MB compressed before upload; > 20MB rejected with clear error. |

#### Module 5: Budget Limit Alerts Module (Real-time & Accuracy)

| KPI Number | KPI Name | Description | Criteria |
| --- | --- | --- | --- |
| KPI-BGT-01 | Alert Delivery Latency | Time between threshold-crossing expense save and alert render. | < 2 seconds. |
| KPI-BGT-02 | Threshold Calculation Accuracy | Correct computation of category spend vs. budget limit. | 100% mathematical accuracy; zero false alerts. |
| KPI-BGT-03 | 80% Soft Alert Trigger | Amber banner displayed when spend hits 80% of category limit. | Alert renders reliably on every threshold crossing. |
| KPI-BGT-04 | 100% Hard Alert Blocking | Red blocking dialog with "Proceed Anyway" confirmation on hard limit breach. | Dialog blocks save until explicit user override. |
| KPI-BGT-05 | Alert Deduplication | No alert spam on rapid successive expenses crossing threshold. | Max 1 consolidated alert per 5-minute window per category. |
| KPI-BGT-06 | Mid-Month Budget Set | Immediate alert if existing spend already exceeds newly set budget. | Alert triggers on budget save, not deferred to next expense. |

#### Module 6: Recurring Expense Auto-Entry Module (Reliability & Scheduling)

| KPI Number | KPI Name | Description | Criteria |
| --- | --- | --- | --- |
| KPI-REC-01 | On-Time Execution Rate | % of scheduled entries created within ±1 hour of billing date. | ≥ 99.5%. |
| KPI-REC-02 | Backfill Reliability | Missed entries (server downtime) auto-created on recovery. | 100% of missed entries backfilled with `source: "auto-backfill"` tag. |
| KPI-REC-03 | Month-End Day Rollover | Correct handling of billing_day=31 in shorter months. | Entry created on last day of month (Feb 28/29, Apr 30, etc.). |
| KPI-REC-04 | Rule Independence | Deleting a generated entry does not cancel the recurring rule. | Rule continues to fire on next scheduled date after entry deletion. |
| KPI-REC-05 | Duplicate Rule Warning | System warns when creating a rule matching existing category+amount. | Warning prompt displayed; user can proceed or cancel. |
| KPI-REC-06 | Notification Delivery | Push notification sent on each auto-entry creation. | Notification delivered within 30 seconds of entry creation. |

#### Module 7: MoM Comparison Chart Module (Visualization & Performance)

| KPI Number | KPI Name | Description | Criteria |
| --- | --- | --- | --- |
| KPI-MOM-01 | Chart Render Time | Render time for multi-month comparison chart. | < 200ms for 12-month dataset. |
| KPI-MOM-02 | Filter Responsiveness | Chart update speed on category filter or date range change. | Chart re-renders with animation in < 300ms. |
| KPI-MOM-03 | Data Accuracy | Displayed values match actual stored expense totals per month. | 100% accuracy; zero rounding discrepancies. |
| KPI-MOM-04 | Insufficient Data State | Graceful handling when < 2 months of data exist. | Single bar shown with message: "Add expenses next month to see trends". |
| KPI-MOM-05 | Zero-Spend Month Display | Months with $0 spending rendered correctly. | Zero-height bar visible (not hidden); tooltip shows "$0 — No expenses". |
| KPI-MOM-06 | Chart Type Toggle | Seamless switch between Bar and Line chart. | Toggle completes with smooth animation in < 200ms; no data loss. |

#### Module 8: Shared Expense Mode Module (Collaboration & Accuracy)

| KPI Number | KPI Name | Description | Criteria |
| --- | --- | --- | --- |
| KPI-SHR-01 | Invite Link Generation | Time to generate and display shareable invite code/link. | < 1 second. |
| KPI-SHR-02 | Join Flow Completion | End-to-end time for invited user to join shared space. | < 30 seconds from link click to shared ledger view. |
| KPI-SHR-03 | Settlement Accuracy | Mathematical correctness of 50/50 split and "who owes whom" calculation. | 100% accuracy; validated by automated test suite. |
| KPI-SHR-04 | Ledger Isolation | Personal ledger entries never leak into shared space and vice versa. | 100% data isolation verified across all CRUD operations. |
| KPI-SHR-05 | Invite Link Security | Link expiry and single-use enforcement. | Link expires after first successful join OR 48 hours, whichever first. |
| KPI-SHR-06 | Duplicate Expense Detection | Flag potential double-entries in shared space. | Prompt shown when same amount + date + category logged by both users. |
| KPI-SHR-07 | Leave Space Graceful Handling | Historical data preserved as read-only for departing user. | Read-only access confirmed; no data deletion on leave. |

---

# KPI Matrix
 
| KPI Number | KPI Name | Description | Criteria |
| --- | --- | --- | --- |
| KPI-AUTH-01 | Login/Register Latency | Measures API response time for authentication requests. | Auth API responses in < 200ms. |
| KPI-CRUD-01 | Input Validation Latency | Measures UI responsiveness to invalid input submission. | Border highlight and warning message display in < 150ms. |
| KPI-CRUD-02 | Database CRUD Latency | Tracks speed of saving/deleting records in PostgreSQL database. | Database updates and state change triggers occur in < 50ms. |
| KPI-CRUD-03 | Form Fields Auto-Reset | Checks if the input form resets after submission. | Form resets to default empty inputs within 100ms of add event. |
| KPI-CHART-01 | Chart Rendering Latency | Measures speed of re-rendering the SVG/Canvas bar chart. | Chart visual re-renders and displays new heights in < 100ms. |
| KPI-CHART-02 | Aggregation Precision | Validates category summation logic. | Categorized totals show zero rounding errors (100% accuracy). |
| KPI-CHART-03 | Chart Empty State Display | Checks if empty chart shows default message. | Zero-expense state renders placeholder text and empty chart graphics. |
| KPI-EXP-01 | Export Completeness | Ensures all records are captured in the CSV. | Number of rows in CSV matches the number of logged expenses. |
| KPI-EXP-02 | CSV Export Speed | Measures latency from button click to download start. | CSV download trigger begins in < 300ms. |
| KPI-EXP-03 | CSV Formula Sanitization | Protects against CSV injection attacks. | All values beginning with `=`, `+`, `-`, or `@` are prepended with a single quote (`'`). |
| KPI-OCR-01 | Scan-to-Save Latency | End-to-end time from image capture/upload to saved expense. | < 8 seconds at p95. |
| KPI-OCR-02 | Amount Extraction Accuracy | % of scans where extracted amount requires no correction. | ≥ 85% accuracy. |
| KPI-OCR-03 | Category Suggestion Accuracy | % of scans where suggested category is accepted. | ≥ 70% accuracy. |
| KPI-OCR-04 | Layout Stability on Processing | Zero layout shift during OCR processing. | CLS = 0 during scan→parse→autofill. |
| KPI-OCR-05 | Error Handling UX | Graceful failure on unreadable images. | Error message in < 2s; form stable. |
| KPI-OCR-06 | Image Size Handling | Client-side compression for large uploads. | > 10MB compressed; > 20MB rejected. |
| KPI-BGT-01 | Alert Delivery Latency | Time between threshold-crossing save and alert. | < 2 seconds. |
| KPI-BGT-02 | Threshold Calculation Accuracy | Correct spend vs. limit computation. | 100% accuracy; zero false alerts. |
| KPI-BGT-03 | 80% Soft Alert Trigger | Amber banner at 80% threshold. | Reliable on every crossing. |
| KPI-BGT-04 | 100% Hard Alert Blocking | Red blocking dialog on hard limit breach. | Blocks save until override. |
| KPI-BGT-05 | Alert Deduplication | No spam on rapid successive expenses. | Max 1 alert per 5-min window. |
| KPI-BGT-06 | Mid-Month Budget Set | Immediate alert if existing spend exceeds new budget. | Alert on budget save. |
| KPI-REC-01 | On-Time Execution Rate | % of entries created within ±1 hour of billing date. | ≥ 99.5%. |
| KPI-REC-02 | Backfill Reliability | Missed entries auto-created on recovery. | 100% backfilled. |
| KPI-REC-03 | Month-End Day Rollover | Correct handling of day 31 in short months. | Last day of month used. |
| KPI-REC-04 | Rule Independence | Deleting entry doesn't cancel rule. | Rule persists. |
| KPI-REC-05 | Duplicate Rule Warning | Warning on matching category+amount rule. | Prompt displayed. |
| KPI-REC-06 | Notification Delivery | Push notification on auto-entry. | Within 30 seconds. |
| KPI-MOM-01 | Chart Render Time | Multi-month chart render speed. | < 200ms for 12 months. |
| KPI-MOM-02 | Filter Responsiveness | Chart update on filter change. | < 300ms with animation. |
| KPI-MOM-03 | Data Accuracy | Values match stored totals. | 100% accuracy. |
| KPI-MOM-04 | Insufficient Data State | Graceful < 2 months handling. | Message shown. |
| KPI-MOM-05 | Zero-Spend Month Display | $0 months rendered correctly. | Zero-height bar + tooltip. |
| KPI-MOM-06 | Chart Type Toggle | Bar ↔ Line switch. | < 200ms animated. |
| KPI-SHR-01 | Invite Link Generation | Time to generate invite code/link. | < 1 second. |
| KPI-SHR-02 | Join Flow Completion | End-to-end join time. | < 30 seconds. |
| KPI-SHR-03 | Settlement Accuracy | 50/50 split correctness. | 100% accuracy. |
| KPI-SHR-04 | Ledger Isolation | Personal vs. shared data separation. | 100% isolation. |
| KPI-SHR-05 | Invite Link Security | Expiry and single-use enforcement. | 48hr or first-join expiry. |
| KPI-SHR-06 | Duplicate Expense Detection | Flag double-entries in shared space. | Prompt on match. |
| KPI-SHR-07 | Leave Space Handling | Read-only access for departing user. | No data deletion. |
 
---
 
# Development Timeline

| Sprint | Focus Area | Deliverables |
| --- | --- | --- |
| Sprint 1 | Core Setup & Auth | React/TypeScript setup, Expense entry UI forms, PostgreSQL DB connection, User Auth APIs, input validation, basic list view. |
| Sprint 2 | Charting & Summary | Charting library integration (Chart.js/Recharts), monthly aggregation logic, dynamic chart updates, responsiveness. |
| Sprint 3 | Export & Polish | CSV download implementation, CSV injection sanitization, UI/UX polish, empty states, QA testing. |
| Sprint 4 | OCR Receipt Scanner | Camera/file upload UI, OCR API integration (Google Vision/Tesseract.js), auto-fill form logic, confidence indicators, error handling. |
| Sprint 5 | Budget Limit Alerts | Budget settings UI, threshold calculation engine, amber/red alert components, hard-limit blocking dialog, alert deduplication. |
| Sprint 6 | Recurring Expenses | Recurring toggle UI, frequency/billing-date selector, server-side scheduler setup, backfill mechanism, notification delivery. |
| Sprint 7 | MoM Analytics Chart | Analytics tab UI, multi-dataset chart (bar/line toggle), category filter, date range selector, tooltip breakdowns. |
| Sprint 8 | Shared Expense Mode | Auth system integration, shared space creation/join flow, shared ledger CRUD, settlement calculation, ledger isolation, invite link security. |
| Sprint 9 | Integration & QA | End-to-end testing across all modules, performance optimization, security audit, UAT. |
 
---
 
# Success Criteria

| Category | Success Metric | Target |
| --- | --- | --- |
| Operational | Core CRUD Capability | 100% successful CRUD actions verified in UAT. |
| Technical | Chart Sync Responsiveness | Real-time chart update within 100ms of adding 5 concurrent test expenses. |
| Compliance | CSV Validation | Exported CSV opens cleanly in Excel/Sheets with correct columns and zero injection vulnerabilities. |
| Usability | Load Time & Smoothness | Initial page load < 1.0s; 0 layout shift during CRUD updates. |
| Accuracy | OCR Field Extraction | ≥ 85% amount accuracy, ≥ 70% category suggestion accuracy across 100-receipt test set. |
| Real-time | Budget Alert Responsiveness | Alerts render within 2 seconds of threshold-crossing expense save in 100% of test cases. |
| Reliability | Recurring Entry Execution | ≥ 99.5% on-time auto-entries; 100% backfill on recovery from downtime. |
| Visualization | MoM Chart Performance | < 200ms render for 12-month dataset; correct values for all months including $0-spend months. |
| Collaboration | Shared Space Data Integrity | 100% ledger isolation; 100% settlement accuracy; invite link expires correctly in all test scenarios. |
| Security | Auth & Invite Flow | Zero unauthorized access to shared spaces; all invite links expire per policy. |
 
---
