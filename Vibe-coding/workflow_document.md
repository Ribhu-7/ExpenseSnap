# Developer Handover & Workflow Document: ExpenseSnap

## 1. Project Overview

* **Project Name**: ExpenseSnap
* **Business Purpose**: Provide freelancers and independent contractors with a streamlined, local-first application to log, track, and export their business expenses without the friction of complex accounts or setups.
* **Problem Statement**: Freelancers often track business expenses manually using physical notebooks, which is prone to errors, provides no visual insights, and makes tax reporting tedious.
* **Solution Summary**: A lightweight, secure web application that allows users to instantly log expense amounts with categories and dates, view monthly spending breakdowns via a dynamic bar chart, and export all records to a CSV file.
* **Key Features**: 
  - Expense Logging (CRUD operations).
  - Real-time Monthly Summary & Dynamic Bar Chart.
  - One-click CSV Export with injection protection.
  - Data persistence via browser LocalStorage.

---

## 2. Technology Stack

| Layer          | Technologies |
| -------------- | ------------ |
| **Frontend**   | React 19, TypeScript, Vite, Tailwind CSS v3, Formik, Yup, Recharts, Lucide React, date-fns, uuid |
| **Backend**    | N/A (Client-side application) |
| **Database**   | Browser LocalStorage (`expense_snap_data`) |
| **Infrastructure** | Node.js (for local dev server), npm |
| **Testing**    | React Testing Library & Jest (Ready to be configured) |

---

## 3. Architecture Overview

* **High-Level Architecture**: Single Page Application (SPA) operating entirely on the client side. Data is managed via React context/hooks and persisted in the browser's LocalStorage.
* **Request/Data Flow**: User Interface (Forms/Buttons) $\rightarrow$ Custom React Hooks (`useExpenses`) $\rightarrow$ Storage Service (`localStorage.ts`) $\rightarrow$ Browser LocalStorage API.
* **Major Components**: `Dashboard` (main view), `ExpenseForm` (data entry), `ExpenseList` (data display), `MonthlySummaryChart` (data visualization).
* **External Integrations**: None. Fully independent and local-first for enhanced privacy.

---

## 4. Project Structure

**Frontend Folder Structure:**
```text
src/
├── charts/          # Visualization components (e.g., MonthlySummaryChart)
├── components/      # Reusable UI components (ExpenseForm, ExpenseList)
├── features/        # Feature-based pages (expenses/pages/Dashboard)
├── hooks/           # Custom React hooks containing business logic (useExpenses)
├── storage/         # Data persistence layer (localStorage)
├── types/           # TypeScript interfaces (Expense, MonthlySummary)
└── utils/           # Helper functions (csvExport)
```
*Briefly explain major folders:*
- **features/**: Groups complex pages by domain (e.g., expenses).
- **hooks/**: Encapsulates state management and local storage syncing so components remain stateless where possible.
- **storage/**: Acts as a mocked backend service, interacting solely with the browser API.

---

## 5. Module Summary

| Module | Purpose | Key Components | Dependencies |
| ------ | ------- | -------------- | ------------ |
| **Dashboard** | Serves as the main layout assembling all sub-modules | `Dashboard.tsx` | `ExpenseForm`, `ExpenseList`, `MonthlySummaryChart` |
| **Expense Logging** | Handles capturing and validating new expense entries | `ExpenseForm.tsx` | Formik, Yup, uuid |
| **Expense Viewing** | Displays existing expenses and allows deletion | `ExpenseList.tsx` | date-fns, Lucide React |
| **Data Visualization** | Renders dynamic spending charts based on category | `MonthlySummaryChart.tsx` | Recharts |
| **Data Export** | Exports data securely to CSV format | `csvExport.ts` | None |

---

## 6. Database Overview

| Entity / Collection | Purpose | Relationships |
| ------------------- | ------- | ------------- |
| `Expense` Array (LocalStorage: `expense_snap_data`) | Stores individual expense records (id, amount, category, date) | Independent |

---

## 7. API Overview

| Endpoint | Method | Purpose | Auth Required |
| -------- | ------ | ------- | ------------- |
| `storage.getExpenses()` | GET (Local) | Fetch all expenses from LocalStorage | No |
| `storage.saveExpenses()` | POST/PUT (Local) | Save/Update the entire expenses array | No |

*Authentication Flow*: As per the PRD, this phase does not require authentication. It operates as a single-user, local-device tool.

---

## 8. Environment & Setup

* **Required Environment Variables**: None required for local development.
* **Installation Steps**: Ensure Node.js (v18+) is installed. Run `npm install` in the project root.
* **Local Setup**: No external databases required. Clone the repository and run the installation.
* **Build Commands**: `npm run build` (Compiles TS and builds Vite optimized bundles).
* **Run Commands**: `npm run dev` (Starts Vite local development server).

---

## 9. Business Workflows

**Workflow 1: Adding a New Expense**
* **Purpose**: Record a business expense.
* **Steps**: User fills out the Form (Amount, Category, Date) $\rightarrow$ Clicks "Add Expense".
* **Validation Rules**: Amount must be positive and $\le$ 999,999.99; Category must not contain special characters (`<>&"'`); Date cannot be in the future.
* **Expected Outcome**: Data is saved to LocalStorage, UI updates immediately (Chart and List reflect new data).

**Workflow 2: Exporting Expenses**
* **Purpose**: Download records for tax or accounting purposes.
* **Steps**: User clicks "Export CSV" on the dashboard.
* **Validation Rules**: Button is disabled if no expenses exist. Output strings starting with `=`, `+`, `-`, or `@` are sanitized with a single quote.
* **Expected Outcome**: A file named `expenses_export.csv` is immediately downloaded to the user's machine.

---

## 10. Security & Error Handling

* **Authentication & Authorization**: None (local-only phase).
* **Sensitive Data Handling**: Data is kept entirely on the user's device. No cloud sync minimizes data breach risks.
* **Validation Strategy**: Client-side validation powered by **Yup**. Prevents malformed data from entering the storage layer.
* **Error Handling Strategy**: UI highlights invalid fields in red. Fallback empty states prevent the application from crashing when local storage is empty or cleared.

| Scenario | Expected Behavior |
| -------- | ----------------- |
| Negative amount entered | Submission blocked, error message "Must be a positive amount" displayed. |
| Malicious script in category | Yup rejects `<>&"'`, preventing basic XSS vulnerabilities. |
| CSV Injection attempt | `csvExport.ts` prepends `'` to leading formula operators. |
| Empty LocalStorage | Application shows "No expenses logged yet" placeholder. |

---

## 11. Testing Overview

| Test Type | Coverage | Tools |
| --------- | -------- | ----- |
| Unit Tests | Planned for hooks and utils | Jest |
| Component Tests | Planned for forms and lists | React Testing Library |
| Manual QA | 100% on CRUD & Chart operations | Browser Developer Tools |

*Test Execution Commands*: Currently planned. To be run via `npm run test` once configured in future sprints.

---

## 12. Deployment & Operations

* **Deployment Process**: Static site generation. The `dist/` folder can be uploaded to any static hosting provider (Vercel, Netlify, GitHub Pages, S3).
* **CI/CD Flow**: Simple pipeline running `npm install`, `npm run lint`, and `npm run build` on push to main branch.
* **Monitoring & Logging**: No external logging is integrated in this phase. Development logs to browser console.
* **Rollback Strategy**: Revert to previous static build deployment on the hosting provider.

---

## 13. Known Limitations & Future Enhancements

| Type | Description | Priority |
| ---- | ----------- | -------- |
| Limitation | Data is bound to the local browser. Clearing cache destroys data. | High |
| Enhancement | Implement Cloud Sync / Backend Database (e.g., Supabase/Firebase) | High |
| Enhancement | User Accounts & Authentication | High |
| Enhancement | Multi-currency support and AI Receipt OCR | Medium |

---

## 14. Troubleshooting Guide

| Issue | Resolution |
| ----- | ---------- |
| Application fails to start (`npm run dev`) | Delete `node_modules` and run `npm cache clean --force`, then `npm install` again. |
| Chart is not rendering properly | Ensure `Recharts` and `React 19` are compatible. Check if container has absolute height defined (e.g., `h-[300px]`). |
| Data is lost on refresh | Ensure browser isn't in strict Incognito mode blocking LocalStorage. Check console for `localStorage.setItem` errors (quota exceeded). |
| TypeScript errors on build | Run `npx tsc --noEmit` to verify type safety. Ensure all models use `import type { Expense } from '../types';` due to Vite verbatim module syntax limits. |

---

## 15. Developer Quick Start

1. **Install dependencies**: `npm install`
2. **Configure environment**: No `.env` setup required for local storage mode.
3. **Run application**: `npm run dev`
4. **Execute tests**: N/A (Manual QA at this phase)
5. **Build project**: `npm run build`
6. **Deploy application**: Copy the contents of the generated `dist/` folder to your static hosting server.

A new developer should be able to start contributing using only this document.
