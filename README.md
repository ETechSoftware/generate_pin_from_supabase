# Subscription Management System

A web-based application for managing subscription PINs, client information, and software licenses with Supabase authentication.

## Project Structure

```
generate_pin_from_supabase/
├── index.html          # Main application (protected, requires login)
├── login.html          # Authentication page (login/signup)
├── users.html          # User management CRUD page
├── tests.txt           # Test data and Supabase SQL notes
└── README.md           # This file
```

## Prerequisites

1. **Supabase Account**: Create a free account at [supabase.com](https://supabase.com)
2. **Supabase Project**: Create a new project and note the:
   - Project URL (e.g., `https://xyz.supabase.co`)
   - anon public key (from Settings > API)

## Setup Instructions

### 1. Create the Supabase Table

Run the SQL commands from `tests.txt` in your Supabase SQL editor:

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

Also add the columns mentioned in `tests.txt`:
```sql
ALTER TABLE pin_id_table 
ADD COLUMN IF NOT EXISTS email VARCHAR(255),
ADD COLUMN IF NOT EXISTS days_status VARCHAR(255),
ADD COLUMN IF NOT EXISTS remaining_days INT;
```

### 2. Configure Supabase Authentication

1. Go to Supabase Dashboard > Authentication > Settings
2. Under "Email settings", configure your email provider
3. Under "Redirect URLs", add `http://localhost:54321/*` (for local development)
4. Enable "Email" authentication provider

### 3. Update Configuration

Edit `index.html` and `login.html` to match your Supabase project:

```javascript
const SUPABASE_URL = 'https://YOUR_PROJECT_REF.supabase.co';
const SUPABASE_ANON_KEY = 'sb_publishable_YOUR_ANON_KEY';
```

## Usage Guide

### 1. Login Page (`login.html`)

- **Login**: Enter email and password to access the main system
- **Signup**: Create a new account (check email for verification link)
- If already logged in, you'll be automatically redirected to `index.html`

### 2. Main Application (`index.html`)

After login, you'll see the Subscription Management System with these tabs:

#### Tabs Overview:

- **All Subscriptions**: View, search, and manage all subscription records
- **Create New**: Add new subscription PINs with client information
- **Search**: Find subscriptions by client, company, email, PIN, or hash
- **Statistics**: View charts and counts of subscription statuses
- **Debug**: Connection info and raw API queries
- **Users**: User management (see below)

#### Key Features:

- **PIN Generation**: Click the dice icon to generate random 6-12 digit PINs
- **PIN Hashing**: SHA-256 hashes generated automatically for security
- **Status Tracking**: Active, Expired, Pending, Trial, Cancelled statuses
- **Remaining Days**: Automatic calculation showing days left until expiration
- **Export to CSV**: Download all records for offline analysis
- **Record Actions**: Edit, view details, or delete subscriptions

#### Creating a New Subscription:

1. Go to "Create New" tab
2. Fill in required fields: Client Name, Company Name, Email, Address, Contact Number
3. Set PIN number (or generate random)
4. Set start date and duration (days/weeks/months/years)
5. Select subscription status
6. Click "Create Subscription"

### 3. User Management (`users.html`)

Accessible via the "Users" tab in the main app (requires authentication).

#### Features:

- **View Current User**: See your session user details
- **Create New User**: Register new users with email and password
- **Password Reset**: Send password reset emails to users
- **Statistics**: Total users, active, pending, unknown counts

#### Note:
User management uses Supabase Auth client APIs. For full admin capabilities (deleting users, etc.), you may need a Supabase service_role key.

## Security Notes

- The application uses Supabase anon key for client-side operations
- PIN hashes are stored using SHA-256 encryption
- Plain PINs are stored separately for display purposes
- Always keep your Supabase anon key secure
- Consider using service_role key for admin operations

## Browser Support

- Chrome, Firefox, Safari, Edge (latest versions)
- Mobile-responsive design for tablets and phones

## Troubleshooting

### Common Issues:

1. **"Connection Failed"**: Check Supabase URL and anon key are correct
2. **"Table not found"**: Ensure `pin_id_table` exists in Supabase with correct schema
3. **"Permission denied"**: Check RLS policies or use service_role key
4. **Login redirect loop**: Clear cookies or check if session is valid
5. **Users tab empty**: User management requires authenticated session

### Debug Tips:

- Use the "Debug" tab to view connection status and run test queries
- Check browser console for JavaScript errors
- Verify Supabase dashboard settings match configuration

## Customization

- Modify styling in the `<style>` sections of each HTML file
- Add more duration units or subscription statuses in the JavaScript
- Extend user management capabilities with Supabase Admin API
- Add more validation rules in the form handling