What it is storing
According to your 

schema.prisma
 file, your database currently stores 6 different tables:

User: Your authentication details (email, securely hashed passwords, name, avatar).
Workspace: Used to separate personal and shared ledgers (includes inviteCode for shared workspaces).
WorkspaceMember: The relationships that map which Users belong to which Workspaces.
Expense: Your core transactions (amount, category, date, and if it was manual vs OCR).
Budget: Your monthly category limits (soft vs hard limits).
RecurringRule: Your scheduled expenses that happen daily, weekly, or monthly.
If you ever want to visually browse or edit the data directly, you can open a new terminal in the expense_snap/backend folder and run npx prisma studio, which will launch a clean visual database editor in your browser!