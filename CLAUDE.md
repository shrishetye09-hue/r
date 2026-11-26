# RBI IT Asset Lifecycle Management System — Project Brief for Claude

Read this whole file before doing anything. It is the single source of truth for this project.
If anything I ask later conflicts with this file, point out the conflict and ask before acting.

---

## 1. Who I am and how you must work

I am a **beginner** developer (basic HTML, CSS, JavaScript, PHP, MySQL). You are my senior software architect, PHP developer, database designer and mentor.

I use **Claude Pro** and must **save usage limits**. Follow these rules strictly:

1. Work **only on the task I ask for**. Never build the whole app or several modules at once.
2. **Never start the next phase or step on your own.** Finish, summarise in 2–3 lines, then wait.
3. Before creating or changing more than 2 files, **list the files and why**, then wait for my "go".
4. When changing an existing file, change **only the needed lines**. Never rewrite a whole file for a small change.
5. Keep replies **short and technical**. No repeated explanations, no long alternatives — pick one good solution.
6. Don't run unnecessary searches, scans or analysis for simple tasks.
7. Reuse existing code and decisions. Don't redesign what is already decided in this file.
8. No paid APIs, AI integrations, or third-party services. Free and open-source only.
9. If a task is big, split it into small steps and do **only step 1**.
10. If something is unclear or a decision is missing, **ask me one short question** instead of guessing.
11. When a decision changes, update **only the affected lines** of this file and tell me in one line what changed.

### Teaching style
- Explain new concepts in simple words, briefly. Don't over-explain basics.
- Tell me exactly **which file/folder**, and give terminal commands only when needed.
- After each step, tell me **how to run and test it** (what URL to open, what I should see).
- When there's an error, explain **why** it happened, then fix it.
- I'm on a **MacBook Air M4 (Apple Silicon, macOS)** and use **VS Code**.

---

## 2. Project goal

One integrated web application for the **IT Department** to manage IT assets through their full life cycle: procurement → registration → allocation → monitoring/maintenance → movement in/out of premises → transfer/recovery → retirement/disposal.

It combines **IT Asset Management**, **IT Asset Movement Monitoring**, **Complaints/Maintenance**, and an **Internal CMS** (RBI employees and departments only — never external customers) into **ONE project with ONE database**.

It will be used **daily by the IT department**, so it must be reliable, secure and simple to use.

**Data rule:** Use **fictional sample data only**. Never request, generate or assume real RBI employee, asset or confidential information. Real deployment needs the bank's IT security approval.

---

## 3. Tech stack (decided — do not change)

| Part | Choice |
|---|---|
| Front end | HTML, CSS, **Bootstrap 5**, plain JavaScript |
| Back end | **Plain PHP 8.x** (no framework) using **PDO + prepared statements** |
| Database | **MySQL 8** (8.0.16+ — needed for CHECK constraints and generated columns) |
| Email | **PHPMailer** (via Composer) through the bank's SMTP server; **Mailpit** for local testing |
| Excel export | **PhpSpreadsheet** (via Composer) |
| Dev environment | **Docker Desktop** (php-apache, mysql, mailpit, phpMyAdmin) |

Keep it beginner-friendly. No Laravel, React, Node build tools, etc., unless I approve.

---

## 4. User roles

| Role | Who | Can do |
|---|---|---|
| **Super Admin** | Application owner | Everything: settings, complaint time limits, categories, audit log, backups |
| **Admin** | IT department staff | Add/deactivate users, manage assets, allocations, fulfil approved requests, complaints, movements, vendor tickets, exports, lifecycle changes |
| **HOD** | Head of a department | Approve/reject asset requests from their own team; view their department's assets |
| **User** | Non-IT employees | Own assets, **plus view and raise complaints on their department's shared assets (cannot change them)**; request assets; log taking an asset out/in; upload photos/documents |

- Every page must check the role (role-based access control).
- An HOD sees **only their own department**.
- Users **never** see other people's personal data.

---

## 5. Login and accounts

- **OTP-only login via email. No passwords anywhere.**
- **Employee Code is the unique ID** for every employee (no duplicates). Email is also unique.
- Only employees **added by an Admin** can request an OTP. Admin (IT team) adds and removes users.
- "Removing" a user = **deactivate** (`is_active = 0`), never delete, so history stays.
- OTP: 6 digits, **expires in 5 minutes**, stored **hashed** (never plain), single use.
- **Lockout is per employee, not per OTP:** after **3 wrong attempts** set `employees.otp_locked_until` (lock time from `settings`). Requesting a new OTP does **not** reset it.
- **Rate limit:** max **3 OTP requests per 15 minutes** per employee (counted from `otp_codes.created_at`).
- Email is **verified on the first successful OTP login** (`email_verified_at`).
- Secure PHP sessions: regenerate session ID on login, timeout on inactivity, CSRF tokens on all forms.
- **Suppliers have NO login and NO access** to the portal.

---

## 6. Modules

| # | Module | What it does | Release |
|---|---|---|---|
| 1 | Asset Management | Register, inventory, allocate (employee / department / common area), transfer/recover, retire/dispose. **An asset used by several people is allocated to the department.** | Phase 1 |
| 2 | Movement Monitoring | Record assets going out of / back into premises with time stamps. Users log their own; Admins verify. | Phase 1 (manual) |
| 3 | Complaints | Complaint per asset with priority, time limit, technician, status. **Reminder + pop-up at 80% of the time limit and when overdue.** | Phase 1 |
| 4 | Vendor Tickets | If IT can't fix a complaint, Admin raises a supplier ticket linked to it. **Sent to the supplier automatically by email.** Supplier replies by email; Admin records each reply in the portal. **Reminder email every 2 days** if no response. Full history kept in `vendor_ticket_updates`. | Phase 1 |
| 5 | Asset History | Repair, replacement, maintenance records per asset | Phase 1 |
| 6 | Internal CMS | Employee profiles, departments, service requests, assets held per employee | Phase 1 |
| 7 | Network Mapping | LAN cable/port ↔ desktop ↔ EPIC No. | Phase 1 |
| 8 | Dashboard | Totals, in use, under repair, pending/overdue complaints with ageing, department-wise count, lost/stolen, pending transfers, open vendor tickets | Phase 1 |
| 9 | Login (OTP) | Section 5 | Phase 1 |
| 10 | Asset Requests | User requests asset → **HOD approves/rejects** → **IT team notified** → IT fulfils | Phase 1 |
| 11 | User Management | IT adds/deactivates users; Employee Code unique | Phase 1 |
| 12 | Uploads | Photos and documents per asset (invoice, warranty card) by users and IT | Phase 1 |
| 13 | Logs & Email | **Every important action is logged**, and emails go out for requests, approvals, complaints, deadlines | Phase 1 |
| 14 | Excel Export | Download lists/reports as Excel | Phase 1 |
| 15 | Listing Views | Searchable, filterable, paginated lists of assets, users, requests, complaints | Phase 1 |
| 16 | Settings | Super Admin edits thresholds and options stored in `settings` | Phase 1 |
| 17 | 3D Floor Model | 3D model of each floor with assets tagged by EPIC No. | Phase 2 |
| 18 | RFID Tagging | Automatic laptop movement tracking at entry/exit points (and possibly metal-detector integration) | Phase 2 |
| 19 | Expiry Notifications | Alerts/emails before warranty or lifespan ends | Phase 2 |
| 20 | Spend Dashboard | IT spend by category and over time, to plan yearly purchases and group requests into bulk orders | Phase 2 |
| 21 | Daily Backup | Automatic daily DB backup, copy kept off the server, restore tested monthly (**suggested to move to Phase 1**) | Phase 2 |

**Do not work on any Phase 2 item until I say so.**

---

## 7. Business rules, lookups and statuses

**Asset categories (lifespan, Super Admin can add/change):** Laptop 36 months · Desktop 60 months · Printer 36 months

**Asset statuses:** `in_stock` → `allocated` → `under_repair` / `out_of_premises` / `lost_stolen` → `retired` → `disposed`
- **Only Admins** can set `lost_stolen`, `retired` or `disposed`.
- Setting any of these **automatically closes the current allocation** (fills `returned_on`).

**Complaint statuses:** `open`, `assigned`, `in_progress`, `pending`, `resolved`, `closed`
- The deadline clock **keeps running** while `pending`.

**Complaint priorities (Super Admin can change):** High 24 h · **Medium 42 h** (to confirm with IT team: 42 or 48) · Low 72 h

**Asset request flow:** `pending_hod` → `approved` / `rejected` (reason required) → `fulfilled`
- Requests from an **HOD, Admin or Super Admin**, or from a department with **no HOD**, skip the HOD step and go straight to `approved`. IT fulfils them.

**Vendor ticket statuses:** `open`, `emailed`, `awaiting_vendor`, `resolved`, `closed`

**Default settings (stored in `settings`):** `near_deadline_percent` = 80 · `vendor_reminder_days` = 2 · `otp_lock_minutes` = 15 · `otp_max_requests_per_15_min` = 3 · `mail_from_address` = (set later)

**EPIC No.** = the number of each compartment/seat. Stored in `locations`.

---

## 8. Asset fields

Inventory No. (unique) · Item Type (category) · Asset Serial No. (unique) · Make · Model · Lifespan (months, copied from category) · Date of Capitalisation · Put to Use Date · EPIC No. (location) · Purchase Cost (₹) · **Date of Expiry = capitalisation date + lifespan (MySQL generated column)** · Warranty End Date · Vendor · Status · Compliance Log · CMS Reference · Stolen/Lost Report · LAN Port/Cable ID (desktops).

**Department and Name of User are NOT stored on the asset.** They come from the current row in `asset_allocations` (`returned_on IS NULL`). This keeps a full transfer history.

---

## 9. Database design (approved and reviewed — Phase 1, 23 tables)

All tables: `id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY`, InnoDB, `utf8mb4`, foreign keys enforced, `created_at` (and `updated_at` where useful) as `DATETIME DEFAULT CURRENT_TIMESTAMP`. Table and column names in `snake_case`.

| Group | Table | Key columns / rules |
|---|---|---|
| People | `departments` | name (unique), code (unique), `hod_employee_id` → employees (**add this FK after `employees` exists** — circular link) |
| People | `employees` | `employee_code` (unique), full_name, email (unique), phone, designation, `department_id`, `role` ENUM('super_admin','admin','hod','user'), `location_id`, `is_active`, `email_verified_at`, `last_login_at`, **`otp_locked_until` DATETIME NULL**, `created_by` |
| People | `otp_codes` | employee_id, `otp_hash`, `expires_at`, `attempts`, `used_at`, ip_address, created_at |
| Settings | `settings` | `setting_key` VARCHAR(100) unique, `setting_value` VARCHAR(255), `updated_by` → employees, updated_at |
| Lookups | `locations` | building, floor, area, `epic_no` (unique), `is_common_area` |
| Lookups | `asset_categories` | name (unique), `lifespan_months`, is_active |
| Lookups | `vendors` | name, contact_person, email (required — tickets go here), phone, address, is_active |
| Lookups | `priority_levels` | name (unique), `time_limit_hours`, sort_order |
| Assets | `assets` | inventory_no (unique), category_id, serial_no (unique), make, model, vendor_id, purchase_cost DECIMAL(12,2), capitalisation_date, put_to_use_date, lifespan_months, **`expiry_date` DATE GENERATED ALWAYS AS (DATE_ADD(capitalisation_date, INTERVAL lifespan_months MONTH)) STORED**, warranty_end_date, location_id, status ENUM, cms_reference, disposal_date, disposal_notes, created_by |
| Assets | `asset_allocations` | asset_id, `allocated_to_type` ENUM('employee','department','common_area'), employee_id / department_id / location_id, allocated_on, `returned_on` (NULL = current holder), allocated_by, request_id. **`is_current` TINYINT GENERATED AS (IF(returned_on IS NULL, 1, NULL)) + UNIQUE(asset_id, is_current)** = only one current holder. **CHECK**: the filled holder column matches `allocated_to_type` (exactly one of the three filled). |
| Assets | `asset_requests` | request_no (unique, e.g. REQ-2026-0001), employee_id, category_id, reason, status ENUM, hod_id, hod_decision_at, hod_remarks, fulfilled_asset_id, fulfilled_by, fulfilled_at |
| Assets | `asset_movements` | asset_id, employee_id, direction ENUM('out','in'), moved_at, purpose, expected_return, logged_by, verified_by, verified_at |
| Assets | `asset_files` | asset_id, file_type ENUM('photo','invoice','warranty','other'), original_name, stored_path, size_kb, uploaded_by, uploaded_at |
| Assets | `network_ports` | port_label (unique), switch_name, location_id, asset_id (unique — one desktop per port) |
| Assets | `lost_stolen_reports` | asset_id, type ENUM('lost','stolen'), incident_date, description, external_report_ref, status ENUM('reported','investigating','recovered','written_off'), reported_by |
| Assets | `compliance_logs` | asset_id, check_type, result ENUM('pass','fail','na'), remarks, checked_by, checked_on |
| Complaints | `complaints` | complaint_no (unique), asset_id, raised_by, description, priority_id, status ENUM, technician_id, **due_at = created + priority hours**, reminder_sent_at, resolution_notes, resolved_at, closed_at. Ageing is calculated, not stored. |
| Complaints | `complaint_updates` | complaint_id, old_status, new_status, note, updated_by |
| Complaints | `vendor_tickets` | ticket_no (unique, in email subject), complaint_id, vendor_id, issue_summary, status ENUM, vendor_ref_no, emailed_at, last_reminder_at, resolved_at, raised_by. **No `vendor_reply` column** — replies live in `vendor_ticket_updates`. |
| Complaints | `vendor_ticket_updates` | ticket_id → vendor_tickets, `direction` ENUM('to_vendor','from_vendor'), message TEXT, recorded_by → employees, created_at |
| Complaints | `maintenance_records` | asset_id, type ENUM('repair','replacement','maintenance'), complaint_id, vendor_id, work_date, description, replaced_part, cost DECIMAL(12,2), done_by |
| Logs | `email_logs` | to_email, employee_id, email_type ENUM, subject (**never store OTPs**), related_table, related_id, status ENUM('queued','sent','failed'), error_message, sent_at |
| Logs | `audit_logs` | `BIGINT` id, employee_id, action, table_name, record_id, old_values JSON, new_values JSON, ip_address. **Never edited or deleted.** |

Phase 2 will add `rfid_tags` / `rfid_reads`.

---

## 10. Security and quality rules (always)

- **Prepared statements** for every query (PDO). Never build SQL from user input.
- **Validate** all input on the server (and on the client for convenience). **Escape** all output with `htmlspecialchars`.
- **CSRF token** on every form. **Role check** at the top of every protected page.
- **Audit log** every create/update/delete/login/export of important data.
- File uploads: **JPG, PNG, PDF only**, max **5 MB**, check real MIME type, random stored filename, stored **outside the public web folder**, served through a PHP script that checks permission.
- Secrets (DB password, SMTP) in a **`.env` / config file outside the web root**, never committed to Git.
- Times stored in the database, displayed in **IST (Asia/Kolkata)**.
- Reusable components (header, sidebar, footer, DB connection, auth check, flash messages).
- Consistent naming: `snake_case` for DB and PHP files, `camelCase` for JS variables.

---

## 11. Proposed folder structure (confirm with me before creating)

```
rbi-asset-system/
├── CLAUDE.md            ← this file
├── docs/                ← requirements spreadsheet, ER diagram (reference only)
├── docker-compose.yml
├── .env                 ← secrets (not in Git)
├── .gitignore
├── composer.json
├── public/              ← web root (only this folder is reachable by browser)
│   ├── index.php
│   ├── assets/          ← css, js, images
│   └── ...pages
├── app/                 ← PHP logic (not public)
│   ├── config/
│   ├── includes/        ← db.php, auth.php, csrf.php, header.php, footer.php
│   ├── controllers/
│   └── helpers/
├── database/
│   ├── schema.sql       ← table creation
│   └── seed.sql         ← fictional sample data
├── storage/
│   ├── uploads/         ← uploaded files (not public)
│   └── logs/
└── vendor/              ← Composer (not in Git)
```

---

## 12. Development phases (build order)

1. ✅ Requirement analysis — **done**
2. ✅ Database design + ER diagram — **done**, reviewed and fixes applied (Sections 5, 7, 9)
3. ⏭ **Development environment on MacBook** (Docker, PHP, MySQL, Mailpit, phpMyAdmin, folder structure, Git) — **NEXT**
4. Login & authentication (OTP, sessions, roles)
5. Asset registration & inventory
6. Employee, department & allocation management (+ user management)
7. Internal CMS + asset requests with HOD approval
8. Complaints, maintenance & vendor tickets (+ reminders, pop-ups, emails)
9. Asset lifecycle: movements, transfers, lost/stolen, retirement, disposal, network mapping
10. Dashboard, listing views, settings page, reports, Excel export
11. Security testing & final improvements (+ daily backup)
12. Phase 2 features (3D floor model, RFID, expiry alerts, spend dashboard) — later

---

## 13. Open questions (ask me only when we reach them)

1. Who approves asset requests when the HOD is absent (a deputy)?
2. Medium priority: 42 or 48 hours?
3. Move daily backup into Phase 1?
4. Does the bank require login through Active Directory instead of email OTP?

---

## 14. Your first reply in a new chat

Do **not** write code or create files yet. Reply with only:
1. "Understood" + a 2–3 line summary of the project.
2. The current phase (Section 12) and what **step 1** of it is, in 2–3 lines.
3. Then **wait** for my go.
