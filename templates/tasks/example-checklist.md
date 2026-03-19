# Monthly Financial Close

**Description:** Standard end-of-month financial reconciliation and reporting
**Creates project:** no
**Variables:** month-name, year
**Default tag:** @finances

## Tasks

- 🔴 High ⏰ Now 🔧 30 min 🕓 60 min ⚡ Admin
  Pull bank statements for {{month-name}} {{year}}
  Context: Download from all accounts — checking, savings, credit cards

- 🔴 High ⏭️ Next 🔧 30 min 🧠 Deep
  Reconcile accounts for {{month-name}}
  Dependencies: Pull bank statements
  Context: Match transactions against records

- 🟡 Medium ⏭️ Next 🔧 20 min ⚡ Admin
  Generate {{month-name}} financial reports
  Dependencies: Reconcile accounts

- 🟡 Medium ⏭️ Next 🔧 15 min ⚡ Admin
  Submit {{month-name}} reports to accounting
  Dependencies: Generate financial reports
