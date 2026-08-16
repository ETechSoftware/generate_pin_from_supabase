# Subscription Management System

A web-based application for managing subscription PINs, client information, software licenses, and system users — powered by [Supabase](https://supabase.com). It runs entirely as static HTML/JS pages, so it can be hosted on any web server (XAMPP/Apache, Nginx, GitHub Pages, etc.).

## Features

- **Authentication** — login/signup with email + password (bcrypt-hashed).
- **Subscription (PIN) management** — create, view, edit, delete, and search subscription records.
- **PIN generation** — one-click random 6–12 digit PIN generator with automatic SHA-256 hashing.
- **Statistics** — status counts and charts (Active, Expired, Pending, Trial, Cancelled, Unknown).
- **Remaining days** — automatic calculation and color-coded display of days left until expiry.
- **CSV export** — download all records for offline analysis.
- **User management** — CRUD for system users (username, email, full name, role, active/inactive).
- **Password tools** — hash a plain-text password into a bcrypt `password_hash`, and verify a password against a stored hash.
- **Role-based access** — admin vs. user permissions (see [Roles & Permissions](#roles--permissions)).

## Roles & Permissions

The `role` column in the `users` table controls access. It must be `admin` or `user`.

| Feature                                    | Admin | User |
|--------------------------------------------|-------|------|
| View subscriptions                         | ✅     | ✅    |
| Search subscriptions                       | ✅     | ✅    |
| View statistics                            | ✅     | ✅    |
| Create subscription (Create New tab)       | ✅     | ❌    |
| Edit subscription                          | ✅     | ❌    |
| Delete subscription                        | ✅     | ❌    |
| User management (Users tab)                | ✅     | ❌    |
| Password tools (hash/verify)               | ✅     | ✅    |

When a non-admin (`user`) logs in, the **Create New**, **Users**, **Edit**, and **Delete** controls are hidden/disabled. The underlying API methods are also guarded, so a regular user cannot trigger them from the browser console.

## Project Structure

```
generate_pin_from_supabase/
├── index.html          # Main application (requires login)
├── login.html          # Login / signup page
├── users.html          # User management + password tools
├── tests.txt           # Scratch notes / test data
└── README.md           # This file
```

---

## 1. Prerequisites

1. **Supabase account** — create a free account at [supabase.com](https://supabase.com).
2. **Supabase project** — create a new project and note your:
   - **Project URL** (e.g., `https://YOUR_PROJECT_REF.supabase.co`)
   - **anon / publishable key** (Dashboard → Settings → API Keys)
3. **A web server** — XAMPP (Apache), Nginx, or any static file server to serve the HTML files.

> The `anon` key is safe to expose in client-side code. The **service_role** key must **never** be used in the browser or committed to the repo.

---

## 2. Supabase Database Setup

Open the **SQL Editor** in your Supabase dashboard and run the following scripts.

### 2.1 Create the `pin_id_table` (subscriptions)

```sql
CREATE TABLE pin_id_table (
    id UUID DEFAULT uuid_generate_v4() PRIMARY KEY,
    client_name TEXT NOT NULL,
    company_name TEXT NOT NULL,
    email VARCHAR(255),
    address TEXT NOT NULL,
    contact_number VARCHAR(255),
    software_start_date DATE NOT NULL,
    software_end_date DATE,
    duration_value INTEGER DEFAULT 1,
    duration_unit VARCHAR(50) DEFAULT 'months',
    subscription_status VARCHAR(50) DEFAULT 'pending',
    pin_number VARCHAR(50) NOT NULL,
    plain_pin VARCHAR(50),
    pin_hash VARCHAR(255),
    payment_name VARCHAR(255),
    reference_number VARCHAR(255),
    last_renewal_date DATE,
    auto_renew BOOLEAN DEFAULT false,
    created_at TIMESTAMP DEFAULT now(),
    updated_at TIMESTAMP DEFAULT now()
);
```

### 2.2 Create the `users` table (accounts)

Used by `login.html` and `users.html` for local (non-Supabase-Auth) accounts.

```sql
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    username TEXT,
    email TEXT UNIQUE NOT NULL,
    password_hash TEXT NOT NULL,
    full_name TEXT,
    role TEXT DEFAULT 'user',
    is_active BOOLEAN DEFAULT true,
    last_login TIMESTAMPTZ,
    created_at TIMESTAMPTZ DEFAULT now(),
    updated_at TIMESTAMPTZ DEFAULT now()
);
```

### 2.3 Enable Row Level Security (RLS)

The app runs with the public **anon** key, so RLS policies must allow the anon role to read and write both tables. Run:

```sql
ALTER TABLE pin_id_table ENABLE ROW LEVEL SECURITY;
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

-- pin_id_table: allow anon full CRUD
CREATE POLICY "anon all pin_id_table" ON pin_id_table FOR ALL USING (true) WITH CHECK (true);

-- users: allow anon full CRUD
CREATE POLICY "anon all users" ON users FOR ALL USING (true) WITH CHECK (true);
```

> **Security note:** These policies are permissive so the app works without a backend. For production, consider tightening policies, using Supabase Auth with row-level ownership, or moving writes behind a backend that uses the `service_role` key.

### 2.4 Insert your first admin account

Create an initial admin so you can log in:

```sql
INSERT INTO users (username, email, password_hash, full_name, role, is_active)
VALUES ('admin', 'YOUR_EMAIL@example.com', 'PLACEHOLDER', 'Administrator', 'admin', true);
```

> `PLACEHOLDER` will not work for login. Either sign up through the app first (then flip the role to `admin` with the SQL below), or use the **Password Tools → Hash Password** page to generate a real bcrypt hash and paste it in:

```sql
UPDATE users SET role = 'admin' WHERE email = 'YOUR_EMAIL@example.com';
```

---

## 3. Supabase Authentication Setup

1. Go to **Authentication → Providers** and make sure **Email** is enabled.
2. Go to **Authentication → URL Configuration**:
   - **Site URL:** `http://localhost`
   - **Redirect URLs:** add `http://localhost/*` (and the domain you will host on).
3. (Optional) Configure your email provider under **Authentication → Settings** if you want signup confirmation emails.

---

## 4. Configure the App

Open `index.html`, `login.html`, and `users.html` and set the Supabase project URL and anon key at the top of each `<script>` block:

```javascript
const SUPABASE_URL = 'https://YOUR_PROJECT_REF.supabase.co';
const SUPABASE_ANON_KEY = 'sb_publishable_YOUR_ANON_KEY';
```

> All three files use the same values. Do **not** put the `service_role` key here.

---

## 5. Run Locally (XAMPP)

1. Copy the project folder into your web root, e.g. `C:\xampp\htdocs\generate_pin_from_supabase\`.
2. Start **Apache** from the XAMPP Control Panel.
3. Open your browser:
   - `http://localhost/generate_pin_from_supabase/login.html`
   - or `http://localhost/generate_pin_from_supabase/index.html` (will redirect to login if not signed in).

You can also serve it with any static server, e.g. `npx serve .` or `python -m http.server`.

---

## 6. Usage Guide

### 6.1 Login / Signup (`login.html`)

- **Sign In** — works two ways:
  1. **Supabase Auth**: if the email/password matches a Supabase Auth account, it signs in via `auth.signInWithPassword()`.
  2. **Users table fallback**: otherwise it checks the `users` table and verifies the bcrypt `password_hash` (also supports legacy plain-text hashes).
- **Create account** — sign up inserts a new row in the `users` table with a bcrypt-hashed password. A new account defaults to `role = 'user'`.

### 6.2 Main Application (`index.html`)

After login you land on the main app. Tabs:

| Tab | Description | Access |
|-----|-------------|--------|
| **All Subscriptions** | Table of all records with status, remaining days, PIN, PIN hash, and action buttons (View / Edit / Delete). | All users (Edit/Delete admin-only) |
| **Create New** | Form to add a subscription record. | Admin only |
| **Search** | Search by client, company, email, PIN, or PIN hash. | All users (Edit admin-only) |
| **Statistics** | Status counts and a chart. | All users |
| **Users** | Link to user management. | Admin only |
| **Debug** | Connection status, table info, and raw query tool. | All users |

#### Creating / Managing a Subscription

1. Go to **Create New**.
2. Fill in **Client Name**, **Company Name**, **Email**, **Address**, **Contact Number** (required), plus optional payment/reference info.
3. **PIN number** — either type one, or click the **dice icon** to generate a random 6–12 digit PIN. The `plain_pin` and `pin_hash` (SHA-256) fields are filled automatically.
4. Set **Start Date** and **Duration** (value + unit: days/weeks/months/years). End date and remaining days are calculated automatically.
5. Choose **Status** (Active, Expired, Pending, Trial, Cancelled, Unknown), optional **Last Renewal Date** and **Auto Renew**.
6. Click **Create Subscription**.
7. To **edit** or **delete**, use the buttons in each row (admin only). Confirm deletion in the modal.

### 6.3 User Management (`users.html`)

Accessible from the **Users** tab (admin only), or directly at `users.html` while signed in.

- **All Users** tab — searchable table of every account with ID, username, email, full name, role, status, last login, created date, and actions:
  - **View** — detail modal.
  - **Edit** — switch to the Create User form pre-filled (Update User). Leave password blank to keep the current one.
  - **Delete** — confirmation modal, then removes the row.
- **Create User** tab — add a user with username, email, password (bcrypt-hashed automatically), full name, role (`user`/`admin`), and active flag.
- **Statistics** tab — total / active / pending / unknown counts.

### 6.4 Password Tools (`users.html` → Password Tools tab)

- **Hash Password** — enter a plain-text password (and choose a bcrypt cost factor, default 10), click **Generate Hash**, and copy the resulting `password_hash`. Paste it into the `users.password_hash` column or use it to seed a user in SQL.
- **Verify Password** — paste a `password_hash` and enter a candidate password. The tool reports **MATCH** or **NO MATCH**, confirming whether a stored hash belongs to a given password.

> Bcrypt hashing runs in the browser using [bcryptjs](https://www.npmjs.com/package/bcryptjs) (loaded from a CDN). A network connection is required to load the CDN libraries on first visit.

---

## 7. Troubleshooting

| Symptom | Likely cause / fix |
|---------|---------------------|
| Page shows "Loading users..." forever / buttons do nothing | A JavaScript error blocked the page. Open DevTools (F12) → Console and check for errors. A common one was a `const supabase` redeclaration clashing with the CDN global — make sure you use `supabaseClient`. |
| "The Supabase library failed to load" | The CDN scripts are blocked. Check internet connection, disable ad-blockers, or mirror the CDN files locally. |
| "Table not found" | Run the SQL in sections [2.1](#21-create-the-pin_id_table-subscriptions) and [2.2](#22-create-the-users-table-accounts). |
| "Permission denied" / rows not loading | RLS policies missing — run section [2.3](#23-enable-row-level-security-rls). |
| Login always fails | Verify the email exists in `users` (or Supabase Auth), the account is `is_active`, and `password_hash` is a valid bcrypt hash (use Password Tools to check). |
| Login redirect loop | Clear browser cookies/localStorage, or confirm you are signed in. |
| Buttons "Create New"/"Users" greyed out | You are logged in as a `user`, not `admin`. Set `role = 'admin'` in the `users` table. |
| Page loads but styling is broken | Bootstrap CSS from the CDN didn't load — check connectivity/ad-blockers. |

### Debug tips

- Use the **Debug** tab to check connection status and run raw queries against `pin_id_table`.
- Open the browser console (F12) to see `[SubscriptionApp]` and `UserManagement:` log lines.
- Verify keys/URL under Supabase Dashboard → Settings → API.

---

## 8. Customization

- **Styling** — edit the `<style>` blocks in each HTML file (colors, fonts, layout).
- **Statuses / duration units** — add options to the `subscription_status` and `duration_unit` dropdowns and the matching JavaScript constants.
- **Role names** — the app treats any role other than `admin` as a regular user; you can add more roles and extend the permission checks in `index.html` and `users.html`.

---

## 9. Important Security Notes

- The **anon (publishable) key** is meant to be public in the browser — this is by design.
- The **service_role key** grants full database access. **Never** put it in HTML/JS or commit it to a repository. If it was ever exposed, rotate it in the Supabase dashboard immediately.
- Passwords are stored as bcrypt hashes in `users.password_hash`. The app never logs plain-text passwords.
- PINs are stored in both `pin_number`/`plain_pin` (plain) and `pin_hash` (SHA-256) for display purposes.
- The RLS policies in section [2.3](#23-enable-row-level-security-rls) allow broad anon access so the app can run without a backend. For stricter environments, restrict them and route writes through a server-side layer using the `service_role` key.