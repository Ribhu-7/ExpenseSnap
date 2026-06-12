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
| **TC-EXP-03** | CSV Formula Sanitization | Verify protection against injection attacks | Application is loaded | 1. Add expense with category starting with `=` (e.g., `=cmd\|' /C calc'!A0`)<br>2. Click "Export CSV" | Exported CSV values beginning with `=`, `+`, `-`, or `@` are safely prepended with a single quote (`'`). |
