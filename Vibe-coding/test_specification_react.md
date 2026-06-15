# Test Specification Document: ExpenseSnap

Based on the Product Requirements Document (PRD) and KPI matrix, the following crisp test cases must be validated for the ExpenseSnap application.

## 1. Expense CRUD Module (Operational & Data Integrity)

| Test Case ID | Feature | Description | Pre-conditions | Test Steps | Expected Outcome (Acceptance Criteria) |
| --- | --- | --- | --- | --- | --- |
| **TC-CRUD-01** | Add Expense | Verify successful addition of a valid expense | Application is loaded | 1. Enter valid amount (e.g., 15.00)<br>2. Enter category (e.g., "Meals")<br>3. Select date<br>4. Click "Add Expense" | Expense is saved to local storage, list updates in < 50ms, and form resets to default empty inputs within 100ms. |
| **TC-CRUD-02** | Form Validation (Empty) | Verify form rejects empty submissions | Application is loaded | 1. Leave fields empty<br>2. Click "Add Expense" | Submission blocked, missing fields highlighted in red with warning message in < 150ms. |
| **TC-CRUD-03** | Form Validation (Negative) | Verify form rejects negative amounts | Application is loaded | 1. Enter amount `-10`<br>2. Click "Add Expense" | Submission blocked, amount field highlighted in red. |
| **TC-CRUD-04** | Limit Validation | Verify amount limit constraint | Application is loaded | 1. Enter amount `1000000`<br>2. Click "Add Expense" | Submission blocked, rejected for exceeding the $999,999.99 limit. |
| **TC-CRUD-05** | Security Sanitization | Verify XSS prevention on inputs | Application is loaded | 1. Enter `<script>` in category field<br>2. Click "Add Expense" | Form rejects input due to invalid special characters. |
| **TC-CRUD-06** | Delete Expense | Verify expense deletion synchronization | At least 1 expense exists | 1. Click "Delete" (trash icon) on an expense | Expense removed from list, local storage updates, chart and totals re-render within 100ms. |

## 2. Monthly Summary & Chart Module (Technical & UI/UX)

| Test Case ID | Feature | Description | Pre-conditions | Test Steps | Expected Outcome (Acceptance Criteria) |
| --- | --- | --- | --- | --- | --- |
| **TC-CHART-01** | Chart Rendering Latency | Verify real-time chart updates | Application is loaded | 1. Add 5 rapid test expenses | Chart visually re-renders and displays new heights in < 100ms. |
| **TC-CHART-02** | Aggregation Precision | Verify category summation logic | Application is loaded | 1. Add two expenses to the same category (e.g., $10.50 + $20.25) | Categorized total dynamically reflects $30.75 exactly (zero rounding errors/100% accuracy). |
| **TC-CHART-03** | Empty State Display | Verify UI on zero expenses | LocalStorage is cleared | 1. Refresh application | UI displays placeholder text ("No expenses logged yet") and empty chart graphics. |

## 3. CSV Export Module (Security & Compliance)

| Test Case ID | Feature | Description | Pre-conditions | Test Steps | Expected Outcome (Acceptance Criteria) |
| --- | --- | --- | --- | --- | --- |
| **TC-EXP-01** | Export Completeness | Verify export contains all records | Multiple expenses exist | 1. Click "Export CSV" | File `expenses_export.csv` downloads. Number of rows matches the number of logged expenses with columns: ID, Amount, Category, Date. |
| **TC-EXP-02** | CSV Export Speed | Verify latency of download trigger | Multiple expenses exist | 1. Click "Export CSV" | CSV download trigger begins in < 300ms (and completes in < 500ms). |
| **TC-EXP-03** | CSV Formula Sanitization | Verify protection against injection attacks | Application is loaded | 1. Add expense with category starting with `=` (e.g., `=cmd|' /C calc'!A0`)<br>2. Click "Export CSV" | Exported CSV values beginning with `=`, `+`, `-`, or `@` are safely prepended with a single quote (`'`). |

## 4. OCR Receipt Scanner Module (Accuracy & Performance)

| Test Case ID | Feature | Description | Pre-conditions | Test Steps | Expected Outcome (Acceptance Criteria) |
| --- | --- | --- | --- | --- | --- |
| **TC-OCR-01** | OCR Parse Coffee | Verify parsing of a coffee receipt file | Application is loaded, OCR Scanner is open | 1. Choose receipt file `starbucks_coffee.jpg`<br>2. Wait for loading overlay | Simulated values extracted: Vendor: "Starbucks", Amount: $7.45, Category: "Meals", Date: today. Skeletons show with zero layout shift. |
| **TC-OCR-02** | OCR Parse AWS | Verify parsing of an AWS/cloud receipt | Application is loaded, OCR Scanner is open | 1. Choose receipt file `aws_hosting.jpg`<br>2. Wait for loading overlay | Simulated values extracted: Vendor: "AWS Cloud", Amount: $143.20, Category: "Software", Date: today. Skeletons show with zero layout shift. |
| **TC-OCR-03** | OCR Parse Error (Screenshot) | Verify rejection of non-receipt screenshot | Application is loaded, OCR Scanner is open | 1. Choose receipt file `screenshot_desktop.png`<br>2. Wait for loading overlay | Displays error: "This doesn't look like a receipt." Form remains stable. |
| **TC-OCR-04** | OCR Parse Error (Large Selfie) | Verify rejection of oversized selfie image | Application is loaded, OCR Scanner is open | 1. Choose file `selfie_photo.jpg` (size > 8MB)<br>2. Wait for loading overlay | Displays error: "OCR_PARSE_FAILED: Could not extract data. Please try with a clearer receipt." |

## 5. Budget Limit Alerts Module (Real-time & Accuracy)

| Test Case ID | Feature | Description | Pre-conditions | Test Steps | Expected Outcome (Acceptance Criteria) |
| --- | --- | --- | --- | --- | --- |
| **TC-BGT-01** | Soft Warning (80%) | Verify amber alert at >=80% budget utilization | Application is loaded, no Meals budget exist | 1. Go to Budgets and set "Meals" budget to $100<br>2. Add "Meals" expense of $80 | Amber alert banner appears at the top indicating 80% spent of $100 limit. |
| **TC-BGT-02** | Hard Warning (100%) | Verify red blocking modal at >=100% budget utilization | TC-BGT-01 completed (spend is $80) | 1. Add "Meals" expense of $25 (total = $105) | Red blocking confirmation modal appears showing proposed breach details and options. |
| **TC-BGT-03** | Hard Override | Verify proceeding past hard block saves expense | TC-BGT-02 modal is open | 1. Click "Proceed Anyway" in the modal | Expense is saved to ledger, monthly totals and chart update, and red warning banner shows in header. |
| **TC-BGT-04** | Mid-Month Budget Alert | Verify alert triggers immediately when setting low budget | Expense of $50 exists under category "Software" | 1. Go to Budgets tab<br>2. Add "Software" budget of $40 | Soft/Hard budget alert triggers immediately on settings save. |

## 6. Recurring Expense Auto-Entry Module (Reliability & Scheduling)

| Test Case ID | Feature | Description | Pre-conditions | Test Steps | Expected Outcome (Acceptance Criteria) |
| --- | --- | --- | --- | --- | --- |
| **TC-REC-01** | Create Recurring Rule | Verify saving expense with recurring flag creates rule | Application is loaded | 1. Fill Expense form: $29.99, Software, check "Schedule as Recurring Cost"<br>2. Choose "Monthly", billing day = 15<br>3. Submit | Rule is created and listed in "Recurring Expenses" section of Budgets tab. |
| **TC-REC-02** | Backfill Scheduling | Verify missed entries are backfilled when scheduler runs | Rule exists with lastRunDate in the past | 1. Trigger scheduler by switching workspace or reloading | Missed entries are automatically generated with source "auto-backfill" tag. |
| **TC-REC-03** | Day Rollover | Verify billing day 31 rolls over to last day of shorter months | Rule exists with billing day = 31 | 1. Run scheduler with date simulated in short month (e.g. Feb) | Entry generated on the last day of the month (Feb 28/29). |

## 7. MoM Comparison Chart Module (Visualization & Performance)

| Test Case ID | Feature | Description | Pre-conditions | Test Steps | Expected Outcome (Acceptance Criteria) |
| --- | --- | --- | --- | --- | --- |
| **TC-MOM-01** | Insufficient Data Placeholder | Verify placeholder state is shown when data is < 2 months | Expenses exist only in 1 month | 1. Go to Analytics tab | UI displays placeholder text "Analytics Insights Pending" and a visual indicator. |
| **TC-MOM-02** | MoM Trend Chart Rendering | Verify chart renders when 2+ months of data exist | Expenses exist in 2+ months | 1. Go to Analytics tab | Bar/Line comparison chart renders totals correctly for both months. |
| **TC-MOM-03** | Category & Range Filters | Verify filtering chart dynamically | TC-MOM-02 completed | 1. Click category filter button (e.g. "Meals")<br>2. Switch range to 3M/12M<br>3. Toggle Chart Type (Bar/Line) | Chart re-renders with animation. Only filtered data is displayed. Toggle operates smoothly. |

## 8. Shared Expense Mode Module (Collaboration & Accuracy)

| Test Case ID | Feature | Description | Pre-conditions | Test Steps | Expected Outcome (Acceptance Criteria) |
| --- | --- | --- | --- | --- | --- |
| **TC-SHR-01** | Invite & Join Space | Verify space invite creation and peer joining | Logged in as Alex | 1. Go to Shared Workspace and click "Create Space"<br>2. Copy code, switch persona to Taylor<br>3. Go to Shared Workspace, enter code and join | Shared Space is connected and active. Workspace dropdown shows Shared Space option. |
| **TC-SHR-02** | Split & Settlement Math | Verify split logic and "Who owes whom" calculation | Shared workspace active | 1. Alex logs $100 expense<br>2. Taylor logs $50 expense<br>3. Open Shared Workspace tab | Settlement summary displays "Taylor owes Alex $25.00". |
| **TC-SHR-03** | Ledger Isolation | Verify personal ledger remains private | Shared workspace active | 1. Switch back to Personal Workspace | Shared expenses do not appear in Personal ledger, maintaining 100% data isolation. |
