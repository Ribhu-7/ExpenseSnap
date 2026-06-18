# PRD:

[prd.md] refer to this file and then do this Role: Senior Product Manager & Technical Architect
Context: I have an existing MVP for an app called "ExpenseSnap". 
Current MVP Scope:
- Target User: Freelancers transitioning from manual notebook tracking.
- Core Features: Simple expense logging (Category + Amount) and a basic monthly totals dashboard.
Objective:
I want to scale this app. Please generate a comprehensive, execution-ready Product Requirement Document (PRD) to add 5 new advanced functionalities. The PRD must include User Stories, Technical Considerations, UI/UX Guidelines, and Edge Cases for each feature.
Here are the 5 functionalities to specify:
1. OCR Receipt Scanner
- User Flow: User takes a photo or uploads an image of a receipt -> App parses data -> Auto-fills 'Amount' and suggests a 'Category'.
- Requirement: User must be able to review and manually edit the auto-filled data before saving.
- UI/UX Note: Maintain layout stability; ensure the keyboard or loading states do not violently shrink or displace the background content during processing.
2. Budget Limit Alerts per Category
- User Flow: User sets a monthly hard/soft cap for specific categories (e.g., $500 for Marketing).
- Requirement: Trigger real-time visual alerts/notifications when spending hits 80% and 100% of the limit.
3. Recurring Expense Auto-Entry
- User Flow: User marks an expense as recurring (Daily, Weekly, Monthly) with a specific billing date.
- Requirement: System automatically injects this entry on the scheduled date without requiring manual app launch.
4. Month-over-Month (MoM) Comparison Chart
- User Flow: A dedicated analytics tab showing visual trends.
- Requirement: A clean, lightweight bar or line chart comparing the current month's spending against previous months, filterable by category.
5. Shared Expense Mode (2-User Split Tracking)
- User Flow: A user invites one peer (e.g., partner, co-founder) via a unique link/code to join a shared space.
- Requirement: 50/50 split tracking. Both users can log expenses to a shared ledger, view who owes whom, and see combined monthly totals while keeping personal ledgers separate.
PRD Output Format Required:
1.  Problem Statement
2. Solution Overview
3. User Flow4. API Design5. Edge Cases6. KPIs (Success Metrics or Acceptance Criterias)7. Limitations , want all the functionalities in one format together dont make different 7 format for each new feature

combine [prd.md] and [prd_v2_advanced_features.md] into one file so that it represents one single prd and not different and include all the features mentioned in both like 1 problem statement should have both files problem statement , main function is to build the website which has both the file's features into one


# KPI:
[prd.md]refer to this update my kpi.md with the new features added

# Scope:
[prd.md][kpi.md] refer to both the files and update my [scope.md]file with the updated features

# Persona:
[prd.md] [kpi.md] refer to this files and update my [persona] folder files accordingly 

# Implementation:
[Agent] refering to all files in this folder make adjustments in the [expense_snap] folder and update the code by updating the features added
The 5 advanced functionalities have been integrated into the code.

Key Implementations
OCR Scanner: Handles image parsing, shows skeleton loaders (satisfying CLS = 0), and autofills the form.
Budget Alerts: Generates soft amber warnings at 80% and red blocking dialogs at 100% requiring override.
Recurring Expenses: Automates entries, supporting rollover logic for shorter months and startup backfills.
Analytics Tab: Renders Line/Bar comparison charts with category filtering and MoM tooltips.
Shared Mode: Supports 2-user magic-link workspaces with 50/50 splits and settlement computations.

# Work Flow
now refer to the [Agent] folder and update my [workflow_document.md] accordingly for the complete workflow of the project


# Testing:
[Agent] refer to this files in this folder and create crisp test cases and also test them
create a test file for testing the project according to the test cases mentioned

# New Updates
[prd.md] refer to this file and add login authentication feature which includes backend db connection as well with postregesql , replace local storage everything with database , and update it accordingly

[prd.md] refer to this file and add login authentication feature which includes backend db connection as well with postregesql , replace local storage everything with database , and update it accordingly

initial file:[prd.md] new file:[prd_v2_advanced_features.md] accordingly give me the[kpi.md] , [scope.md] [persona] version 2 which were in prdv2

same way [prd.md] refer to this file and make necessary changes in [kpi.md] , [scope.md] [persona] folders

[Agent]refering to this folder kindly create a database persona as well inside [persona]folder

refering to the [Agent] folder now create the authentication feature in [expense_snap] and properly connect frontend backend database and build and run the project

give me the full walkthrough of running db , backend and frontend for starting and running the project

do the testing of the authentication because it is not working, fix it 

test the project as shared expenses page is not opening please fix it 

test the entire project by creating test files and update the test cases status in [test_execution_report.md]

create test cases and test file for testing the authentication as well and add status in the test_execution_report there as well , there is error in setup.ts fix that as well

client:525 Failed to create shared space SyntaxError: Unexpected token '<', "<!DOCTYPE "... is not valid JSON when creating shared workspace please fix it