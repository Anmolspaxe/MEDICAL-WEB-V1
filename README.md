# 🏥 MediCare General Hospital — Web Application

A comprehensive hospital management web application built with **pure HTML, CSS, and JavaScript** — no frameworks, no build tools. Powered by **Supabase** for the database and authentication, and **EmailJS** for automated email delivery.

---

## 🌐 Live Pages

| Page | File | Access |
|------|------|--------|
| Home | `page1_home.html` | Public |
| Services | `page2_services.html` | Public |
| Doctors | `page3_doctors.html` | Public |
| About Us | `page4_aboutus.html` | Public |
| Reviews | `page5_reviews.html` | Public |
| Book Appointment | `page6_appointment.html` | Public |
| Login / Signup | `login.html` | Public |
| Reset Password | `reset-password.html` | Public |
| Doctor Dashboard | `page8_doctor_dashboard.html` | Doctor only |
| Patient Portal | `page10_patient_portal.html` | Patient only |
| Reports Portal | `page9_reports.html` | Doctor + Staff |
| Reception Panel | `Reception_admin_panel.html` | Receptionist only |

---

## ✨ Features

### 👤 Patient
- Register / Sign in with email and password
- Book appointments by selecting doctor, date, and time
- Receive automatic confirmation email on booking
- View all appointments with status and confirmed time slot
- View medical reports (X-Ray, MRI, Ultrasound, etc.)
- View prescriptions written by doctors
- Forgot password / reset via email link

### 🗂️ Receptionist
- Register with staff code + Sign in
- View all appointments in real-time dashboard
- Accept appointments → assign time slot → auto-email patient
- Decline appointments → auto-email patient
- Book appointments manually on behalf of patients
- Upload patient reports (images, PDFs) to Supabase Storage
- Filter by status, date, doctor

### 🩺 Doctor
- Register / Sign in (gets unique ID like `MCR-CARD-4821-RS`)
- View only their own patient appointments
- See confirmed time slots assigned by receptionist
- Write prescriptions with medicines, dosage, frequency
- Save prescription + email it directly to patient
- View and resend past prescriptions
- View patient reports uploaded by staff

---

## 🛠️ Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | HTML5, CSS3, Vanilla JavaScript |
| Database | Supabase (PostgreSQL) |
| Auth | Supabase Auth (email + password) |
| Storage | Supabase Storage (patient reports) |
| Email | EmailJS (no backend needed) |
| Fonts | Google Fonts — DM Sans, Syne, DM Serif Display |

---

## 🗄️ Database Schema

Run the following SQL in your Supabase **SQL Editor** to set up all tables:

```sql
-- ── APPOINTMENTS ──────────────────────────────────────────────
CREATE TABLE appointments (
  id               uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  patient_id       text,
  name             text NOT NULL,
  email            text NOT NULL,
  phone            text,
  gender           text,
  age              int,
  appointment_date date NOT NULL,
  time             time NOT NULL,
  subject          text,
  doctor           text,
  reason           text,
  status           text DEFAULT 'pending',
  confirmed_time   text,
  created_at       timestamptz DEFAULT now()
);
ALTER TABLE appointments DISABLE ROW LEVEL SECURITY;
ALTER PUBLICATION supabase_realtime ADD TABLE appointments;

-- ── DOCTORS ───────────────────────────────────────────────────
CREATE TABLE doctors (
  id          uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  auth_id     uuid REFERENCES auth.users(id),
  doctor_id   text UNIQUE NOT NULL,
  name        text NOT NULL,
  email       text UNIQUE NOT NULL,
  department  text NOT NULL,
  phone       text,
  created_at  timestamptz DEFAULT now()
);
ALTER TABLE doctors DISABLE ROW LEVEL SECURITY;

-- ── RECEPTIONISTS ─────────────────────────────────────────────
CREATE TABLE receptionists (
  id          uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  auth_id     uuid REFERENCES auth.users(id),
  staff_id    text UNIQUE NOT NULL,
  name        text NOT NULL,
  email       text UNIQUE NOT NULL,
  phone       text,
  created_at  timestamptz DEFAULT now()
);
ALTER TABLE receptionists DISABLE ROW LEVEL SECURITY;

-- ── PATIENTS ──────────────────────────────────────────────────
CREATE TABLE patients (
  id          uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  auth_id     uuid REFERENCES auth.users(id),
  patient_id  text UNIQUE NOT NULL,
  name        text NOT NULL,
  email       text UNIQUE NOT NULL,
  phone       text,
  age         int,
  gender      text,
  created_at  timestamptz DEFAULT now()
);
ALTER TABLE patients DISABLE ROW LEVEL SECURITY;

-- ── PRESCRIPTIONS ─────────────────────────────────────────────
CREATE TABLE prescriptions (
  id             uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  appointment_id uuid,
  patient_name   text NOT NULL,
  patient_id     text,
  patient_email  text NOT NULL,
  doctor_name    text NOT NULL,
  doctor_id      text NOT NULL,
  diagnosis      text NOT NULL,
  medicines      text,
  instructions   text,
  rx_date        date NOT NULL,
  followup_date  date,
  email_sent     boolean DEFAULT false,
  created_at     timestamptz DEFAULT now()
);
ALTER TABLE prescriptions DISABLE ROW LEVEL SECURITY;

-- ── PATIENT REPORTS ───────────────────────────────────────────
CREATE TABLE patient_reports (
  id            uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  patient_name  text NOT NULL,
  patient_id    text,
  report_type   text NOT NULL,
  doctor_name   text NOT NULL,
  report_date   date NOT NULL,
  notes         text,
  file_path     text,
  uploaded_by   text,
  uploaded_role text,
  created_at    timestamptz DEFAULT now()
);
ALTER TABLE patient_reports DISABLE ROW LEVEL SECURITY;
```

---

## 🗂️ Supabase Storage Setup

1. Go to **Storage** in your Supabase dashboard
2. Click **New Bucket**
3. Name it: `patient-reports`
4. Enable **Public bucket** → Create

---

## 🔐 Supabase Auth Setup

1. Go to **Authentication → Providers → Email** — ensure it's **enabled**
2. Go to **Authentication → URL Configuration** and add your site URL:
   ```
   https://your-domain.com/reset-password.html
   ```
3. Optional — disable email confirmation for development:
   **Authentication → Settings → Disable email confirmations**

---

## 📧 EmailJS Setup

1. Create a free account at [emailjs.com](https://emailjs.com)
2. Connect your Gmail under **Email Services** → copy **Service ID**
3. Create a template under **Email Templates** with these variables:
   ```
   To: {{to_email}}
   Subject: Appointment Confirmation - MediCare Hospital
   Body: {{message}}
   ```
   Copy the **Template ID**
4. Go to **Account** → copy your **Public Key**
5. Fill in these values in `page6_appointment.html` and `Reception_admin_panel.html`:
   ```js
   const EMAILJS_SERVICE  = 'service_xxxxxxx';
   const EMAILJS_TEMPLATE = 'template_xxxxxxx';
   const EMAILJS_PUBLIC   = 'xxxxxxxxxxxxxxx';
   ```

---

## ⚙️ Configuration

All files share the same two Supabase credentials. Search for these constants across all HTML files and replace with your own:

```js
const SUPABASE_URL      = 'https://your-project.supabase.co';
const SUPABASE_ANON_KEY = 'your-anon-key';
```

The **staff registration code** for receptionist signup is set in `login.html`:
```js
const STAFF_CODE = 'MCRHOSPITAL2024'; // change this to something secret
```

---

## 🚀 Getting Started

1. **Clone the repo**
   ```bash
   git clone https://github.com/your-username/medicare-hospital.git
   cd medicare-hospital
   ```

2. **Set up Supabase**
   - Create a project at [supabase.com](https://supabase.com)
   - Run the SQL from the Database Schema section above
   - Create the `patient-reports` storage bucket
   - Enable email auth and set the redirect URL

3. **Configure credentials**
   - Replace `SUPABASE_URL` and `SUPABASE_ANON_KEY` in all HTML files
   - Replace EmailJS keys in `page6_appointment.html` and `Reception_admin_panel.html`

4. **Open in browser**
   ```
   Open page1_home.html in any browser — no build step needed
   ```

---

## 📁 Project Structure

```
medicare-hospital/
├── page1_home.html              # Public homepage with chat widget
├── page2_services.html          # Hospital services
├── page3_doctors.html           # Doctors showcase
├── page4_aboutus.html           # About the hospital
├── page5_reviews.html           # Patient reviews
├── page6_appointment.html       # Book appointment form
├── login.html                   # Unified login/signup (all roles)
├── reset-password.html          # Password reset page
├── page8_doctor_dashboard.html  # Doctor personal dashboard
├── page9_reports.html           # Medical reports portal
├── page10_patient_portal.html   # Patient health portal
├── Reception_admin_panel.html   # Receptionist admin panel
└── README.md
```

---

## 🔄 System Flow

```
Patient  ──► Book Appointment ──► Supabase (pending)
                                        │
Receptionist ──► Accept + Assign Slot ──► Supabase (accepted + confirmed_time)
                                        │
                               📧 Email sent to patient with slot
                                        │
Doctor ──► View Appointment ──► Write Prescription ──► Save to Supabase
                                        │
                               📧 Email prescription to patient
                                        │
Receptionist ──► Upload Report ──► Supabase Storage + patient_reports table
                                        │
Patient ──► Patient Portal ──► View appointments + reports + prescriptions
```

---

## 🆔 Unique ID Formats

| Role | Format | Example |
|------|--------|---------|
| Doctor | `MCR-{DEPT}-{4 digits}-{initials}` | `MCR-CARD-4821-RS` |
| Receptionist | `REC-{4 digits}` | `REC-0042` |
| Patient | `MCP-{year}{4 digits}` | `MCP-20241234` |

---

## 📬 Email Triggers

| Event | Recipient | Content |
|-------|-----------|---------|
| Appointment booked | Patient | Confirmation + Patient ID |
| Appointment accepted | Patient | Confirmed time slot |
| Appointment declined | Patient | Decline notification |
| Prescription written | Patient | Full prescription details |

---

## 🔒 Access Control

| Page | Doctor | Receptionist | Patient | Public |
|------|--------|-------------|---------|--------|
| Public pages (1–5) | ✅ | ✅ | ✅ | ✅ |
| Book Appointment | ✅ | ✅ | ✅ | ✅ |
| Doctor Dashboard | ✅ | ❌ | ❌ | ❌ |
| Reception Panel | ❌ | ✅ | ❌ | ❌ |
| Reports Portal | ✅ | ✅ | ❌ | ❌ |
| Patient Portal | ❌ | ❌ | ✅ | ❌ |

---

## 📝 License

MIT License — free to use, modify, and distribute.

---

## 👨‍💻 Author

Built by **Anmol** — MediCare General Hospital Web Application  
Powered by Supabase + EmailJS + Vanilla JS
