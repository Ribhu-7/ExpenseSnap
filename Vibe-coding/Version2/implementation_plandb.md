# Implement Authentication & PostgreSQL Backend

We are transitioning the `expense_snap` app from a local-storage MVP to a full-stack application with user authentication and a PostgreSQL database.

## User Review Required

> [!IMPORTANT]
> **PostgreSQL Database:** To connect to PostgreSQL, we need a running database instance. 
> - Do you have PostgreSQL installed locally on your Mac? 
> - If so, could you provide the connection string (e.g., `postgresql://user:password@localhost:5432/expensesnap`)? 
> - Alternatively, we can use Docker to spin up a local instance, or use a cloud provider like Supabase/Neon. How would you like to proceed with the database hosting for this development phase?

> [!WARNING]
> **Data Migration:** Replacing local storage means any existing expenses currently stored in your browser will be ignored by the new system unless we build a migration script. We will start with a fresh database for authenticated users. Let me know if you need a local-storage-to-database migration path.

## Proposed Changes

We will split the application into two parts: a `backend` folder for the Node/Express server, and the existing React code will serve as the `frontend`.

---

### Backend (Node.js + Express + Prisma)

#### [NEW] `expense_snap/backend/package.json`
Initialize a new Node.js project with `express`, `cors`, `pg`, `prisma`, `bcrypt`, `jsonwebtoken`, and `dotenv`.

#### [NEW] `expense_snap/backend/prisma/schema.prisma`
Define the database schema with Prisma. We'll create models for `User`, `Workspace`, `Expense`, `Budget`, and `RecurringRule`.

#### [NEW] `expense_snap/backend/src/server.ts`
Set up the Express server, CORS, JSON body parser, and base routing.

#### [NEW] `expense_snap/backend/src/routes/auth.ts`
Implement `/api/auth/register` and `/api/auth/login` endpoints with JWT generation and bcrypt password hashing.

#### [NEW] `expense_snap/backend/src/routes/expenses.ts`
(And corresponding routes for Budgets/Recurring). Implement REST endpoints to replace local storage logic. 

---

### Frontend (React + Vite)

#### [MODIFY] `expense_snap/src/context/AuthContext.tsx`
Remove local storage persistence. Implement logic to check for a JWT token, and make `login` and `register` API calls to the new backend.

#### [MODIFY] `expense_snap/src/storage/localStorage.ts`
Replace the local storage read/write methods with HTTP requests (`fetch` or `axios`) to the backend APIs. (We may rename this file to `api.ts`).

#### [NEW] `expense_snap/src/components/AuthPage.tsx`
Create a new UI component for Login and Registration since one does not currently exist. 

#### [MODIFY] `expense_snap/src/App.tsx`
Implement a routing layer or state toggle to show the `AuthPage` if the user is not authenticated, protecting the main dashboard.

---

## Verification Plan

### Automated Tests
- Verify that backend authentication endpoints return standard JWT tokens and appropriate 401/403 errors.

### Manual Verification
- Start the PostgreSQL database.
- Run the backend server (`npm run dev` in the backend folder).
- Run the frontend server (`npm run dev` in the frontend folder).
- Open the application, register a new user, log in, and add an expense.
- Verify the expense is saved to PostgreSQL and not just `localStorage`.
