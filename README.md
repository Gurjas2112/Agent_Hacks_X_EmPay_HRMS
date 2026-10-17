<div align="center">

# EmPay – Smart Human Resource Management System

**A full-stack HRMS in PHP 8 and MySQL. It covers role-based access, NFC and geofenced attendance, leave workflows, payroll and payslips, bulk CSV onboarding and automated email notifications.**

![PHP](https://img.shields.io/badge/PHP-8.2-777BB4?logo=php&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL%20%2F%20MariaDB-PDO-4479A1?logo=mysql&logoColor=white)
![Tailwind CSS](https://img.shields.io/badge/Tailwind%20CSS-3-06B6D4?logo=tailwindcss&logoColor=white)
![Leaflet](https://img.shields.io/badge/Leaflet-1.9.4-199900?logo=leaflet&logoColor=white)
![PHPMailer](https://img.shields.io/badge/PHPMailer-SMTP-EA4335?logo=gmail&logoColor=white)
![Composer](https://img.shields.io/badge/Composer-CI-885630?logo=composer&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green)

Built by **Team Agent Hacks** for the [Odoo × VIT Pune Hackathon 2026](https://hackathon.odoo.com/event/odoo-x-vit-pune-hackathon-26-18/register)

🎬 **[Watch the prototype video](app_prototype_video/EMPAY_HRMS_PROTOTYPE_VIDEO.mp4)** (stored with Git LFS)

</div>

---

## 📌 TL;DR for Reviewers

| | |
|---|---|
| **What it is** | An HR platform covering employees, attendance (manual, GPS-geofenced and NFC kiosk), leave, shift schedules, payroll and payslips, reports, and email automation |
| **Stack** | PHP 8.2 (no framework) · MySQL/MariaDB via PDO · Tailwind CSS · Vanilla JS · Leaflet.js · Web NFC API · PHPMailer · Composer · GitHub Actions |
| **Scale** | 14 relational tables with foreign keys, generated columns and 3 SQL views · 24 frontend pages · 17 backend handlers · 4 roles |
| **Notable engineering** | Front-controller router with path sanitisation · Haversine geofencing · Web NFC "tap & go" kiosk · payroll totals computed by the database · role-based SMTP with delivery audit log · CSV import with duplicate detection |
| **Run it** | XAMPP, then `composer install`, then import `config/empay_hrms.sql`. See [Quick Start](#-quick-start) |

---

## 📑 Table of Contents

1. [Problem Statement](#-problem-statement)
2. [Key Features](#-key-features)
3. [System Architecture](#️-system-architecture)
4. [Tech Stack](#-tech-stack)
5. [Database Design](#️-database-design)
6. [Payroll Model](#-payroll-model)
7. [Role-Based Access Control](#-role-based-access-control)
8. [Attendance Pipelines (Manual · Geo · NFC)](#-attendance-pipelines)
9. [Email Automation](#-email-automation)
10. [Project Structure](#-project-structure)
11. [Quick Start](#-quick-start)
12. [Demo Credentials](#-demo-credentials)
13. [Engineering Highlights](#-engineering-highlights)
14. [Roadmap & Known Gaps](#-roadmap--known-gaps)
15. [Team](#-team)

---

## 🎯 Problem Statement

Many small and mid-sized organisations still manage HR across spreadsheets, email threads and paper registers. The hackathon challenge was to build a working **Human Resource Management System** with:

1. **User & Role Management**: separate permission tiers for Admin, HR Officer, Payroll Officer and Employee.
2. **Attendance & Leave**: daily attendance tracking and a leave approval workflow.
3. **Payroll Management**: salary records with statutory deductions (PF, Professional Tax, TDS) and payslips.
4. **Dashboard & Analytics**: an at-a-glance view of workforce health.

EmPay covers all four and adds **NFC kiosk attendance**, **GPS geofencing with a live map**, **designation-based work and leave policies**, **bulk CSV onboarding** and **automated email notifications**.

---

## ✨ Key Features

### 👥 Employee Management
- Employee directory with search, department filters and live headcounts in the sidebar.
- Create and edit profiles with department, designation, salary, date of joining, date of birth, gender and address.
- **Auto-generated login credentials**: a new employee gets a password in the pattern `EMP{id}2002` (bcrypt-hashed) and an HTML welcome email.
- **Bulk CSV import**: upload a file, columns are mapped by header name, department and designation names are resolved to IDs, and duplicate emails or usernames are skipped. A summary screen then lets HR **choose which new employees get welcome emails**.

### ⏱️ Attendance (Three Capture Channels)
| Channel | How it works |
|---|---|
| **Web check-in/out** | One-click clock in/out with a live clock; one record per user per day, enforced by a `UNIQUE(user_id, date)` key |
| **Geofenced check-in** | Browser Geolocation API sends coordinates; the server uses the **Haversine formula** to classify the check-in as `office` or `remote` against a configurable office radius |
| **NFC kiosk** | `public/nfc_attendance.php` is a full-screen **Web NFC** (`NDEFReader`) page: tap a card to check in, tap again to check out, then it resets for the next person |

- **Late detection**: check-ins after **09:15** are marked `late`.
- **Work hours** are a **MySQL generated column** (`TIMESTAMPDIFF` of check-in and check-out).
- **Live attendance map** (Admin/HR) built with **Leaflet.js**, showing the office geofence circle and a marker with details for each employee's check-in location.
- Personal attendance history plus an organisation-wide log for Admin, HR and Payroll.

### 🌴 Leave Management
- Five leave types (Casual, Sick, Annual, Unpaid, Maternity) with max days, paid/unpaid and carry-forward flags.
- Employees apply with a date range and reason. Day count is computed and end-before-start dates are rejected.
- **HR is emailed** when a request is submitted; the **employee is emailed** when it is approved or rejected.
- Approver ID and timestamp are recorded for the audit trail.
- **Per-year leave balances** with `remaining_days` as a generated column.

### 📅 Shift Scheduling
- Admin/HR assign dated shifts (start, end, notes) to any employee. The employee gets a **styled email** with the shift details.
- All roles can view the schedule; only Admin/HR can create or delete shifts.

### 💰 Payroll & Payslips
- Monthly payroll record per employee: basic, HRA, transport and special allowance, minus PF, professional tax, TDS and other deductions.
- **Gross, total deductions and net salary are computed by the database** as `GENERATED ALWAYS … STORED` columns, so they always match their components.
- Status lifecycle **Draft → Generated → Paid**. Marking a record *Paid* stamps `paid_on` and **emails the employee**.
- **Printable payslip** page with an earnings and deductions breakdown. Employees can open **only their own** payslips (checked on the server).
- **Email a formatted HTML payslip** to an employee in one click.

### 📊 Dashboard & Reports
- **Role-aware dashboard**:
  - Admin, HR and Payroll see total employees, present today, on leave today, pending approvals, recent attendance and recent leave requests.
  - Employees see their own status and history.
- **Email delivery panel**: sent and failed counts with an SVG ring chart, plus the latest delivery logs.
- **Reports page** showing headcount, pending leave and paid payroll totals, plus a one-click **emailed workforce analytics report** broken down by department.

### ⚙️ Administration
- **Designations** manager.
- **Work policies** per designation: working days per week, weekly offs, start and end time.
- **Leave policies** per designation: paid, sick and casual quotas.
- Both policy types **fall back to a default** when a designation has no policy of its own.
- **Geo settings**: set office latitude, longitude and geofence radius (stored in `system_settings`).

---

## 🏗️ System Architecture

```
                       ┌───────────────────────────────────────┐
                       │                Browser                │
                       │  Tailwind UI · Lucide icons · Leaflet │
                       │  Geolocation API · Web NFC (NDEF)     │
                       └──────────────────┬────────────────────┘
                                          │ HTTP(S) (Apache / ngrok tunnel)
                                          ▼
┌───────────────────────────────────────────────────────────────────────────┐
│ public/                                                                   │
│  index.php ──► front controller: config → session → demo provisioner     │
│      │                                                                    │
│      ▼                                                                    │
│  router.php ──► ?page=module/view → sanitise → frontend/<module>/<view>   │
│                                                                           │
│  nfc_attendance.php ── standalone kiosk (no login, tap-to-log)            │
├───────────────────────────────────────────────────────────────────────────┤
│ auth/      session.php · login_check.php (30-min idle timeout)            │
│            role_check.php (requireRole, canManageUsers/Payroll/Leaves)    │
├───────────────────────────────────────────────────────────────────────────┤
│ frontend/  dashboard · users · attendance · leave · schedule · payroll    │
│            reports · admin (designations, policies, settings, geo)        │
│            └── components/ header · navbar · sidebar (role-filtered nav)  │
├───────────────────────────────────────────────────────────────────────────┤
│ backend/   POST handlers (Post/Redirect/Get + flash messages)             │
│            auth · users · attendance · leave · schedule · payroll ·       │
│            reports · attendance_geo.php (JSON API)                        │
├───────────────────────────────────────────────────────────────────────────┤
│ config/    app.php · database.php (PDO singleton) · mail.php ·            │
│            policies.php · geo_helpers.php (Haversine, settings)           │
│ utils/     mailer.php (PHPMailer + email_logs) · db_init.php (self-heal)  │
└──────────────────────────────┬────────────────────────────────────────────┘
                               │ PDO (prepared statements)
                               ▼
                   ┌────────────────────────┐        ┌──────────────────┐
                   │  MySQL / MariaDB       │        │  Gmail SMTP      │
                   │  empay_hrms (14 tbl,   │        │  (role-based     │
                   │  3 views)              │        │   sender accts)  │
                   └────────────────────────┘        └──────────────────┘
```

**Request flow:** reads (GET) go through `index.php?page=…` to a **frontend view**. Writes (POST) go straight to a **backend handler**, which validates input, checks the role, runs prepared SQL, sets a flash message and redirects. This is the **Post/Redirect/Get** pattern, so refreshing the page never resubmits a form.

---

## 🧰 Tech Stack

| Layer | Technology | Purpose |
|---|---|---|
| **Language** | PHP 8.2 (no framework; uses `match`, typed signatures, variadics) | Routing, business logic, rendering |
| **Database** | MySQL 8 / MariaDB 10.4+ (InnoDB, utf8mb4) | Relational storage, FK constraints, generated columns, views |
| **Data access** | PDO with real prepared statements (`EMULATE_PREPARES = false`) | Protection against SQL injection |
| **Styling** | Tailwind CSS (CDN) with custom design tokens (`brand #714B67`) + `custom.css` | Consistent UI styled after Odoo |
| **Icons / Fonts** | Lucide Icons, Google Fonts (Inter, Outfit) | Visual language |
| **Maps** | Leaflet.js 1.9.4 | Live attendance map and geofence picker |
| **Device APIs** | Geolocation API, Web NFC (`NDEFReader`) | Geo check-in, NFC kiosk |
| **Email** | PHPMailer over Gmail SMTP (STARTTLS :587) | Transactional notifications |
| **Dependencies** | Composer (`phpmailer`, `dompdf`, `google2fa`, `phpdotenv`, `monolog`) | Package management |
| **CI** | GitHub Actions, *PHP Composer* workflow | Validates `composer.json` / `composer.lock` and installs dependencies on every push or PR |
| **Dev tooling** | XAMPP, phpMyAdmin, ngrok (`composer tunnel`) | Local hosting and public demo URL for mobile NFC/GPS testing |

---

## 🗄️ Database Design

Full dump: [`hrms/config/empay_hrms.sql`](hrms/config/empay_hrms.sql) · Annotated base schema: [`hrms/config/database.sql`](hrms/config/database.sql)

```
departments ─┐                       ┌── designations ──┬── work_policies
             │ 1:N               N:1 │                  └── leave_policies
             ▼                       ▼
           ┌──────────────── users ────────────────┐
           │ 1:N        │ 1:N       │ 1:N    │ 1:N  │ 1:N
           ▼            ▼           ▼        ▼      ▼
      attendance     leaves    leave_      payroll  schedules
      (geo, status,  (approved_ balances   (generated
       work_hours*)   by → users) (remaining*) gross/net*)
                        │
                   leave_types

   activity_log · email_logs · system_settings      (* = GENERATED column)
```

| Table | Highlights |
|---|---|
| `users` | Unique `username` and `email`; bcrypt `password`; `role` ENUM; FK to department and designation; indexed by role, department and active flag |
| `attendance` | `UNIQUE(user_id, date)`; **generated** `work_hours`; `status` ENUM (present/late/absent/half_day/on_leave); `latitude`, `longitude`, `location_type`, `ip_address` |
| `leaves` | Date range, `days DECIMAL(4,1)` (supports half days), status workflow, FK `approved_by → users` |
| `leave_balances` | `UNIQUE(user, type, year)`; **generated** `remaining_days` |
| `payroll` | `UNIQUE(user_id, month)`; **generated** `gross_salary`, `total_deductions`, `net_salary` |
| `work_policies` / `leave_policies` | Per-designation rules with an `is_default` fallback |
| `email_logs` | Every outbound email with sender, recipient, subject, sent/failed status and error message |
| `system_settings` | Key-value config (office coordinates, geofence radius, NFC mode) |
| Views | `v_employees`, `v_attendance_summary` (monthly present/late/absent and average hours), `v_payslips` |

Referential actions are deliberate: deleting a user **cascades** to their attendance, leave and payroll rows, while approver and creator references use **SET NULL** so the audit history survives.

---

## 💵 Payroll Model

Net pay is computed by the **database**, not by application code, so the stored totals can never disagree with their components:

```sql
gross_salary     = basic_salary + hra + transport + special          -- GENERATED
total_deductions = pf + professional_tax + tds + other_deductions    -- GENERATED
net_salary       = gross_salary − total_deductions                   -- GENERATED
```

Conventions used in the seeded payroll data:

| Component | Rule |
|---|---|
| HRA | 25% of basic |
| Transport | ₹3,000 flat |
| Provident Fund (PF) | Entered per employee (3.6% of basic in the seed data) |
| Professional Tax | ₹200 / month |
| TDS | Entered per employee |

**Example (April 2026, Arjun Mehta):** basic ₹60,000 + HRA ₹15,000 + transport ₹3,000 + special ₹5,000 = **gross ₹83,000**
− PF ₹2,160 − PT ₹200 − TDS ₹3,040 = **net ₹77,600**

---

## 🔐 Role-Based Access Control

Access is enforced **on the server for every request**, not only by hiding menu items:

- `login_check.php` rejects unauthenticated requests and enforces a **30-minute idle timeout**.
- `requireRole(...)` restricts a whole page or handler. Helper functions (`canManageUsers()`, `canManagePayroll()`, `canManageLeaves()`) control individual actions.
- Ownership checks stop employees acting on other people's records (e.g. marking someone else's attendance or opening someone else's payslip).

| Capability | Employee | HR | Payroll | Admin |
|---|:---:|:---:|:---:|:---:|
| Dashboard (scoped to role) | ✅ own | ✅ org | ✅ org | ✅ org |
| View employee directory | ✅ | ✅ | ✅ | ✅ |
| Add / edit employees, CSV import | ❌ | ✅ | ❌ | ✅ |
| Designations, work and leave policies | ❌ | ✅ | ❌ | ✅ |
| Mark own attendance / view own history | ✅ | ✅ | ✅ | ✅ |
| Organisation-wide attendance log | ❌ | ✅ | ✅ | ✅ |
| Live attendance map | ❌ | ✅ | ❌ | ✅ |
| Apply for leave | ✅ | ✅ | ✅ | ✅ |
| Approve / reject leave | ❌ | ✅ | ✅ | ✅ |
| Assign / delete shifts | ❌ | ✅ | ❌ | ✅ |
| Payroll management, email payslips | ❌ | ❌ | ✅ | ✅ |
| View own payslips | ✅ | ✅ | ✅ | ✅ |
| Reports | ❌ | ✅* | ✅ | ✅ |
| System and geo settings | ❌ | ❌ | ❌ | ✅ |

<sub>*HR can open the reports page by URL; the sidebar link is shown to Admin and Payroll.</sub>

---

## 📡 Attendance Pipelines

```
① Web check-in                ② Geofenced check-in                 ③ NFC kiosk
─────────────                  ──────────────────────               ───────────
POST mark_attendance.php       navigator.geolocation                NDEFReader.scan()
  │ owner/role check             │ (lat, lng)                          │ onreading → serialNumber
  │ one row per day              ▼                                     ▼
  │ late if > 09:15            fetch → attendance_geo.php (JSON)    GET nfc_attendance.php?uid=…
  ▼                              │ Haversine(user, office)             │ lookup by id / username
INSERT / UPDATE attendance       │ ≤ radius → 'office' else 'remote'   │ no row today → check in
                                 ▼                                     │ open row     → check out
                               INSERT attendance (+ lat/lng)           │ closed row   → "already logged"
                                                                       ▼
                                                                    full-screen result, auto-reset in 5 s
```

The Haversine distance is computed in [`config/geo_helpers.php`](hrms/config/geo_helpers.php):

```php
$angle = 2 * asin(sqrt(sin($Δlat/2)**2 + cos($lat1) * cos($lat2) * sin($Δlng/2)**2));
return $angle * 6371000; // metres
```

---

## 📧 Email Automation

All mail goes through a single function, `sendEmPayEmail($to, $subject, $html, $role)`, in [`utils/mailer.php`](hrms/utils/mailer.php):

- **Role-based sender identities**: `hr`, `payroll` and `system` each have their own SMTP account and display name.
- **Every attempt is logged** to `email_logs` with its status and error, and shown in the dashboard's email panel.
- The function calls `initDatabase()` first, which creates `email_logs` and `system_settings` if they are missing (**self-healing schema**).

| Trigger | Recipient | Sender role |
|---|---|---|
| Employee created | New employee (credentials) | `hr` |
| CSV import → "Send account emails" | Selected new employees | `system` |
| Leave applied | HR inbox | `system` |
| Leave approved / rejected | Employee | `hr` |
| Shift assigned | Employee | `system` |
| Payroll marked *Paid* | Employee | `payroll` |
| "Email payslip" | Employee | `payroll` |
| "Email report" | Management | `payroll` |

---

## 📁 Project Structure

```
Agent_Hacks_X_EmPay_HRMS/
├── .github/workflows/php.yml        # CI: composer validate --strict + install
├── app_prototype_video/             # Demo video (Git LFS)
├── migrate_roles_to_designations.sql   # Schema migration: roles → designations
├── cleanup_designations_demo.sql
├── test_workflow_with_credentials.md
└── hrms/
    ├── public/                      # ── Web root ──
    │   ├── index.php                # Front controller
    │   ├── router.php               # ?page= router with sanitisation + 404
    │   ├── nfc_attendance.php       # NFC kiosk (Web NFC)
    │   ├── setup_demo_data.php · seed_map_data.php · check_data.php
    │   └── assets/css/custom.css
    ├── auth/                        # session · login_check · role_check
    ├── components/                  # header · navbar · sidebar · footer
    ├── frontend/                    # ── Views (24 pages) ──
    │   ├── dashboard/  users/  attendance/ (mark, history, log, map)
    │   ├── leave/  schedule/  payroll/ (index, payslip, my_payslips)
    │   ├── reports/  auth/
    │   └── admin/ (designations, work_policies, leave_policies, settings, geo_settings)
    ├── backend/                     # ── POST handlers / JSON API ──
    │   ├── auth/  users/ (create, update, import_csv, send_welcome_emails)
    │   ├── attendance/  attendance_geo.php
    │   ├── leave/ (apply, approve)  schedule/ (assign, delete)
    │   ├── payroll/ (generate_salary, update_status, email_demo_payslip)
    │   └── reports/ (email_demo_report)
    ├── config/                      # app · database · mail · policies · geo_helpers · *.sql
    ├── utils/                       # mailer · db_init · demo_provisioner
    ├── employees1–6.csv             # Sample files for bulk import
    ├── composer.json / composer.lock
    └── workflow.md                  # Extra technical notes
```

---

## 🚀 Quick Start

### Prerequisites
- **XAMPP** (Apache + PHP 8.1+ + MariaDB/MySQL) or any LAMP/WAMP stack
- **Composer**
- *(Optional)* Gmail account with an **App Password** for email features
- *(Optional)* Android Chrome for Web NFC, and **ngrok** for HTTPS, which Geolocation and NFC require on phones

### 1 · Clone into the web root

`BASE_URL` in `hrms/config/app.php` expects the app at `/nfc-office-erp/hrms/public/`, so clone into a folder with that name:

```bash
cd C:/xampp/htdocs                       # or /var/www/html
git clone https://github.com/Gurjas2112/Gurjas2112-Agent_Hacks_X_EmPay_HRMS.git nfc-office-erp
cd nfc-office-erp/hrms
composer install
```

> If you clone into a different folder, update the path in `BASE_URL` in `config/app.php`.

### 2 · Create the database

```bash
mysql -u root -e "CREATE DATABASE empay_hrms CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci"
mysql -u root empay_hrms < config/empay_hrms.sql
```

Or create `empay_hrms` in **phpMyAdmin** and import `config/empay_hrms.sql`. Connection settings are in `config/database.php` (default `root`, empty password).

### 3 · Configure email (optional)

Edit `config/mail.php` with your own SMTP account and app password for the `hr`, `payroll` and `system` roles.

### 4 · Launch

Open **http://localhost/nfc-office-erp/hrms/public/** and sign in with a demo account.

| Page | URL |
|---|---|
| App | `…/hrms/public/index.php` |
| NFC kiosk | `…/hrms/public/nfc_attendance.php` (`?uid=<user id>` simulates a tap) |
| Seed map demo data | `…/hrms/public/seed_map_data.php` |

### 5 · Public demo URL (optional)

```bash
composer tunnel        # runs: npx ngrok http 80
```

---

## 🔑 Demo Credentials

| Role | Email | Password |
|---|---|---|
| **Admin** | `admin@empay.com` | `admin123` |
| **HR Officer** | `hr@empay.com` | `hr123` |
| **Payroll Officer** | `payroll@empay.com` | `pay123` |
| **Employee** | `emp@empay.com` | `emp123` |

- Employees imported from CSV get the temporary password `Empay@123`.
- Employees created through the form get `EMP{id}2002` by email.

**5-minute walkthrough**
1. **Admin**: go to *Geo Settings* and set the office location. Then open *Employees → Bulk Import* and upload `hrms/employees1.csv`.
2. **Employee**: *Mark Attendance*, check in with location, then *Apply Leave*.
3. **HR**: *Manage Leaves* and approve the request (the employee gets an email). Then *Schedule* and assign a shift.
4. **Admin/HR**: *Attendance Map*, to see today's check-ins against the geofence.
5. **Payroll**: *Payroll*, open a payslip, mark it *Paid*, then print or email it.
6. Open `nfc_attendance.php?uid=3` twice to simulate an NFC check-in and check-out.

---

## 🏆 Engineering Highlights

- **Framework-free MVC-style layering.** Front controller, router, views, handlers, config and utilities are clearly separated, and a new module is just a new folder.
- **Database does the arithmetic.** Generated columns for work hours, leave balance and payroll totals remove a whole class of consistency bugs.
- **Defence in depth.**
  - Prepared statements everywhere, with emulation off.
  - bcrypt hashing via `password_hash`.
  - Router whitelist regex plus `..` stripping.
  - Server-side role and ownership checks.
  - Session idle timeout.
  - Output escaping with `htmlspecialchars` in views and emails.
- **Integrity in the schema.** Unique keys stop double check-ins, duplicate payslips per month and duplicate accounts. Foreign keys with deliberate `CASCADE` / `SET NULL` rules protect related data.
- **Hardware integration from a browser.** Web NFC and Geolocation work without a native app, using a geofence that admins can configure.
- **Observable side effects.** Every email is audited in `email_logs`, and the dashboard shows the delivery health.
- **Batch UX.** CSV import reports skipped duplicates by name, then lets HR choose who receives credentials.
- **CI on every push.** GitHub Actions validates `composer.lock` and the dependency install.

---

## 🧭 Roadmap & Known Gaps

Known gaps, listed openly:

| Area | Current state | Planned |
|---|---|---|
| Payroll generation | `generate_salary.php` is a stub; payroll rows come from seed data and the demo provisioner | Compute monthly payroll from `attendance` + `leave_policies` (proration, PF/PT slabs, TDS) |
| PDF payslips | Browser print and HTML email | Server-side PDF with **dompdf** (already a dependency) |
| Self-registration | The register form lets a user pick any role | Force `employee` on sign-up; only Admin assigns elevated roles |
| Demo login fallback | Hard-coded demo accounts bypass the DB for presentations | Put behind an `APP_ENV=demo` flag |
| Secrets | DB and SMTP credentials live in PHP config files | Load from `.env` via **phpdotenv** (already a dependency) and rotate the committed keys |
| CSRF | Forms have no CSRF tokens | Per-session token middleware |
| NFC identity | Card UID maps to user `id` / `username` | Dedicated `nfc_tag` column and card enrolment UI |
| Deployment | `BASE_URL` path is hard-coded | Derive from the request or the environment |
| 2FA / logging | `google2fa` and `monolog` installed but not yet used | TOTP for Admin/Payroll; structured logs |
| Tests | Manual QA | PHPUnit suite run by the existing CI workflow |

---

## 👥 Team

**Team Agent Hacks**: [Odoo × VIT Pune Hackathon 2026](https://hackathon.odoo.com/event/odoo-x-vit-pune-hackathon-26-18/register)

| | Name | Role | GitHub |
| :---: | --- | --- | --- |
| <img src="https://github.com/Gurjas2112.png" width="60"> | **Gurjas Singh Gandhi** | Team Leader | [@Gurjas2112](https://github.com/Gurjas2112) |
| <img src="https://github.com/joyboy-pega.png" width="60"> | **Joy Kujur** | Developer | [@joyboy-pega](https://github.com/joyboy-pega) |
| <img src="https://github.com/sarveshvarode092704.png" width="60"> | **Sarvesh Varode** | Developer | [@sarveshvarode092704](https://github.com/sarveshvarode092704) |
| <img src="https://github.com/prathamesh-coding.png" width="60"> | **Prathamesh Nibandhe** | Developer | [@prathamesh-coding](https://github.com/prathamesh-coding) |

> An earlier version built as an **Odoo 19 module with a Flask client** is kept in the project history.

---

## 📄 License

MIT. See `composer.json`.

<div align="center"><sub>EmPay: Simplifying HR & Payroll Operations for Smarter Workplaces</sub></div>
