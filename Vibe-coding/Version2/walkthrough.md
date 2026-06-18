# Full-Stack Authentication & PostgreSQL Backend

We have successfully migrated ExpenseSnap from a local-storage frontend-only MVP to a full-stack, database-backed application featuring secure authentication.

## What Was Achieved

1. **PostgreSQL Database**
   - Launched a PostgreSQL 15 instance via Docker.
   - Initialized Prisma and pushed a fully normalized, relational schema (`Users`, `Workspaces`, `Expenses`, `Budgets`, etc.).
2. **Node.js/Express Backend**
   - Created the `/api/v1/auth` endpoints for Registration, Login, and fetching user data.
   - Implemented JWT generation and bcrypt password hashing.
   - Added `/api/v1/expenses` endpoints with complete CRUD operations.
   - Built an authentication middleware (`authenticateToken`) to protect all API routes.
3. **Frontend Integration**
   - Built a sleek `AuthPage` component for Login and Signup flows.
   - Updated `App.tsx` to conditionally render the Dashboard only if the user is authenticated.
   - Refactored `AuthContext.tsx` and `useExpenses.ts` to utilize native `fetch` requests with JWT Authorization headers instead of reading from `localStorage`.
   - Setup Vite proxy to route `/api` calls smoothly to the backend at port 3001.

## How to Verify
Both servers are currently running in the background. You can navigate to the ExpenseSnap application at `http://localhost:5173/`. 

> [!TIP]
> You will now be greeted by the new Sign Up / Sign In screen. 
> 
> 1. Create a new account using the form.
> 2. You will be successfully logged in and taken to the Dashboard.
> 3. Add an expense.
> 4. If you look at the backend logs or query the PostgreSQL database directly, you will see your expense securely saved in the database!

## Next Steps
Currently, `useBudgets` and `useRecurring` are still using `localStorage`. We can convert those over to the backend APIs next to achieve a 100% database-driven application. Let me know how you'd like to proceed!
