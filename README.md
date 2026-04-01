# Iqrar
Digital Acknowledgment Letter Signing System


> A full-stack web system that digitizes the entire acknowledgment letter workflow — from admin letter management and group assignment through to legally-traceable employee signatures — live in production since September 2025.

---

## Project Overview

Paper-based acknowledgment letters are slow to distribute, hard to track, and impossible to audit at scale. I built **Iqrar** to replace that process entirely — a role-based web system where admins manage and assign letters to employees, and employees sign them digitally using one of three signature methods, all with real-time database updates and a full audit trail.

The system went from first line of code to live production in under three weeks.


---

## My Role

I was the **lead developer** on this project — responsible for the full stack: database schema design, PHP backend, front-end dashboard UI, file handling, PDF signing pipeline, server deployment, and all debugging through to production. I also iterated the design from a single-page layout into a full multi-section dashboard based on real feedback during development.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Backend | PHP |
| Database | MySQL |
| PDF signing | FPDI / FPDF |
| File transfer & deployment | FileZilla |
| Server | Apache (Linux) |
| Frontend | HTML, CSS, JavaScript, jQuery (AJAX) |
| Signature capture | Canvas-based e-signature (browser) |
| OTP authentication | Taqnyat API |

---

## How It Works

### System architecture

```
Admin side                          User side
──────────────────                  ──────────────────
Login (admin)                       Login (employee PRN)
    ↓                                       ↓
Admin Dashboard                     My Letters Dashboard
    ├── Manage admins               View assigned letters
    ├── Manage groups               Choose signature method
    ├── Upload / select letters          ├── E-signature (canvas)
    └── Assign to user or group          ├── Digital stamp
                ↓                        └── OTP (Taqnyat API)
        MySQL updated                           ↓
        Assignment created              Signature saved as PNG
                                        FPDI overlays onto PDF
                                        Signed PDF stored in DB
                                        Assignment marked acknowledged
```

---

## Part 1 — Admin System

Admins have a full management dashboard with four core areas:

### Admin management
- Activate or deactivate other admin accounts
- Role-based access — only active admins can log in and manage the system

### Group management
- Create named groups combining employees by department or any custom criteria
- Add or remove members from groups
- Deactivate groups when no longer needed
- Searchable employee lookup when building groups

### Letter management
- Upload new PDF letters directly through the dashboard
- Or select from previously uploaded letters stored in MySQL
- Set the required signature type per letter: e-signature, digital stamp, or OTP — with manual signature as default

### Assignment
- Assign a letter to an individual employee or an entire group in one action
- Assign the same letter to multiple recipients simultaneously
- View all assignments with live status: **Acknowledged** or **Not Acknowledged**
- Letters automatically sorted: pending first, acknowledged below

---

## Part 2 — The Three Signature Methods

This was the most technically complex part of the system. Each signature method follows a different path but ends at the same place: a signed PDF stored in the database.

### 1. E-signature
The employee draws their signature directly on an HTML canvas in the browser. On submission:
- The canvas image is captured and sent to PHP
- Saved as a PNG file in `uploads/signatures/`
- FPDI opens the original PDF, overlays the signature image at the correct position
- The signed PDF is saved and the assignment record updated in MySQL

### 2. Digital signature
A stamp is automatically generated containing the employee's name and the date and time of signing — no drawing required. The stamp is overlaid onto the PDF using the same FPDI pipeline.

### 3. OTP signature
- The employee's phone number is retrieved from the `Unified_Employees_MasterList` database table
- A one-time password is sent to that number via the **Taqnyat API**
- The employee enters the OTP to confirm their identity
- Verification triggers the same PDF signing and database update pipeline

---

## Part 3 — Database Design

Six MySQL tables power the system, each with a distinct responsibility:

| Table | Purpose |
|---|---|
| `DoCatriona_Admin` | Admin accounts and activation status |
| `DoCatriona_AdminGroup` | Admin-managed group membership (add / remove) |
| `DoCatriona_Assignment` | Letter assignments per employee, with status and signed PDF path |
| `DoCatriona_groups` | Group definitions including member lists |
| `DoCatriona_letters` | Uploaded letters and metadata |
| `Unified_Employees_MasterList` | Source of truth for employee data including phone numbers |

---

## Part 4 — Production Deployment & Problem Solving

Getting this system to production on an Apache server involved solving several real infrastructure problems:

**Apache file write permissions** — PHP scripts run as the `apache` user, not the deployment user. Signed PDFs and signature images couldn't be saved until directory ownership was transferred from `ucas_build` to `apache` and write permissions were correctly set.

**Duplicate POST submissions on refresh** — after assigning a letter, refreshing the page resent the POST data, creating duplicate assignments. Solved by restructuring the flow to redirect after POST (Post/Redirect/Get pattern).

**Headers already sent error** — including `manage_group.php` before `assign_to.php` in the main admin file caused HTML to be output before PHP could call `header()`. Solved by moving all form-handling logic to the top of each included file before any HTML output, and converting form submissions to AJAX to avoid full-page reloads entirely.

**SQL column omission bug** — a server-level 500 error traced back to `Group` and `assigned_by` columns missing from a SELECT query. Added reserved-word backtick quoting for `Group` to prevent future MySQL parsing issues.

---

## Screenshots

| Employee dashboard | E-signature | OTP verification | Document Signed Stamp
|---|---|---|---|
| ![Iqrar_Employee_dashboard_clean](https://github.com/user-attachments/assets/e2a8b129-af5a-4900-9eb1-e27e2276f0ef)| ![Iqrar_E-Sign_clean_1](https://github.com/user-attachments/assets/54fe9a61-d8a9-420f-9536-c9f9df8c1016) | ![Iqrar_OTP_clean](https://github.com/user-attachments/assets/565734c4-8927-4d14-bf7e-5b271c1e1d00) | ![Iqrar_Signature_Stamp_clean](https://github.com/user-attachments/assets/e4ce4d20-a2ca-47e0-8b1f-a076a2077de0)


---

## What I Learned

**PHP architecture discipline** — early in the project, mixing HTML output with form-handling logic caused cascading header errors. Separating logic from presentation (PHP processing at the top, HTML output below) and moving to AJAX submissions fixed the problem cleanly and made the codebase significantly easier to maintain.

**Server-level debugging** — Apache permission errors, ownership mismatches, and directory write failures don't produce useful PHP errors — they show up as 500s or silent failures. Learning to diagnose at the server level, not just the code level, was a major shift in how I approach production issues.

**PDF manipulation in PHP** — overlaying a canvas-captured signature image onto an existing PDF using FPDI while preserving the original document required understanding how FPDI handles page imports, coordinate systems, and image placement. The pipeline had to work correctly for all three signature types.

**Iterative design under a deadline** — the system started as a single-page layout and evolved into a full multi-section collapsible dashboard during development. Delivering that redesign while also fixing production bugs on a three-week timeline sharpened my ability to prioritize and scope changes effectively.

---

## Contact

Built by **Danah Bawajeeh** — http://linkedin.com/in/danahwbawajeeh
