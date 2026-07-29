# ShebaConnect

ShebaConnect is a citizen-services web platform for Bangladesh that lets citizens file and track complaints, apply for government services, find nearby service offices, book appointments, and receive SMS/WhatsApp notifications — with an admin console for managing services, reviewing applications, and viewing analytics built from public/open-data sources.

## Tech Stack

**Frontend (`client/`)**
- React 19 + Vite
- React Router v7
- Tailwind CSS
- Recharts (analytics charts), Framer Motion (animation)
- jsPDF + jspdf-autotable (PDF report generation)
- Axios

**Backend (`server/`)**
- Node.js + Express 5
- MongoDB + Mongoose
- JWT-based authentication, bcryptjs for password hashing
- Multer (file uploads), node-cron (scheduled jobs)
- Twilio (SMS/WhatsApp notifications), Nodemailer (email)
- Google Maps Services JS (geolocation for nearby offices)
- Hugging Face Inference API (AI-assisted complaint drafting & Bangla translation)

## Project Structure

```
sheba-connect/
├── client/                        # React SPA
│   └── src/
│       ├── pages/                 # Route-level views (citizen + admin)
│       ├── pages/admin/           # Admin-only views
│       ├── components/            # Shared UI, analytics widgets, modals
│       ├── components/analytics/  # Analytics dashboard widgets
│       ├── services/              # API clients (general + analytics)
│       ├── config/                # Axios instance & API base config
│       └── utils/                 # PDF/report generation helpers
├── server/                        # Express API
│   ├── controllers/               # Route handlers
│   ├── routes/                    # Express routers
│   ├── models/                    # Mongoose schemas
│   ├── middleware/                # Auth & admin guards, upload handling
│   ├── services/                  # SMS, calendar, weather, World Bank, data sync
│   ├── jobs/                      # Cron jobs (reminders, public-data sync)
│   ├── config/db.js               # MongoDB connection
│   └── server.js                  # App entry point
├── requirements.txt                # Reference list of package versions
├── ANALYTICS_FEATURE_EXTENSION.md  # Notes on the public-data analytics extension
└── .gitignore
```

> Note: `client/server/` is a leftover duplicate copy from earlier development and is not part of the running application. The live backend is `server/` at the project root.

## Core Features

- **Authentication & Roles** — JWT-based login/registration with citizen and admin roles, protected routes on the client (`PrivateRoute`) and role checks on the API (`adminMiddleware`).
- **Complaints** — Citizens file complaints against departments, track status (Pending/Processing/Resolved) through a timeline, receive admin feedback/questions, and edit submissions with a tracked edit history.
- **AI-Assisted Complaint Drafting** — Optional Hugging Face-powered complaint generation and English↔Bangla translation to help citizens draft formal complaints.
- **Service Applications** — Citizens browse available government services, apply for a service, and submit supporting documents; admins review and process applications.
- **Document Management** — Upload, store, and retrieve supporting documents (IDs, forms, proofs) tied to a citizen's profile or a specific application.
- **Appointments** — Citizens book appointments at service offices; includes reschedule requests and admin appointment management.
- **Nearby Offices** — Location-based lookup of nearby service offices using Google Maps geolocation.
- **Notifications** — In-app notifications plus SMS/WhatsApp alerts (via Twilio or Alpha SMS) for status updates and appointment reminders, sent through scheduled cron jobs.
- **Surveys** — Citizen feedback/survey collection (e.g. post-service satisfaction).
- **Solutions/Recommendations** — Suggested solutions and recommendations surfaced to citizens or admins.
- **Admin Console** — Service management, application review, department complaint boards, and system-wide stats/reports (with PDF export).
- **Public Data Analytics Dashboard** — Pulls and caches data from external Bangladeshi open-data APIs (geo divisions/districts/upazilas, World Bank indicators, weather) on a schedule, and visualizes it alongside internal service metrics (resolution times, complaint volume, etc.).

## Data Model (MongoDB via Mongoose)

Key collections: `User`, `Complaint`, `ServiceApplication` / `Application`, `Service`, `Office`, `Appointment`, `Document` / `UserDocument`, `Notification`, `Survey`, `Solution`, `Recommendation`, `Helpline`, `ResolutionTime`, plus analytics-support models `publicData`, `geoData`, `analytics`, `weatherData`, and `worldBankData`. See `server/models/` for full schema definitions.

## API Overview

All routes are mounted under `/api` on the Express app. Protected routes require a `Bearer` JWT; admin-only routes are additionally gated by `adminMiddleware`.

| Base Path | Purpose |
|---|---|
| `/api/auth` | Registration and login |
| `/api/complaints` | File, view, update, and track complaints |
| `/api/applications`, `/api/service-applications` | Service applications and their review |
| `/api/services` | Browse available government services |
| `/api/offices` | Service office directory / nearby lookup |
| `/api/appointments` | Book, reschedule, and manage appointments |
| `/api/documents` | Upload and retrieve supporting documents |
| `/api/notifications` | In-app notifications |
| `/api/sms` | SMS/WhatsApp notification sending |
| `/api/surveys` | Survey submission and retrieval |
| `/api/solutions` | Suggested solutions |
| `/api/helplines` | Helpline directory |
| `/api/users` | User profile and stats |
| `/api/stats`, `/api/reports` | System-wide statistics and report generation |
| `/api/analytics` | Public open-data analytics (geo, weather, World Bank) |
| `/api/admin`, `/api/iftiadmin` | Admin-only management endpoints |
| `/api/ai` | AI-assisted complaint drafting and translation |

## Getting Started

### Prerequisites
- Node.js 18+
- A MongoDB connection string (local or Atlas)
- (Optional) Twilio credentials for SMS/WhatsApp notifications
- (Optional) Hugging Face API token for AI complaint drafting/translation
- (Optional) Google Maps API key for nearby-office geolocation

### 1. Clone and install
```bash
git clone <repo-url>
cd sheba-connect

# install backend deps
cd server && npm install

# install frontend deps
cd ../client && npm install
```

### 2. Configure environment variables
Copy `server/.env.example` to `server/.env` and fill in the values:
```
PORT=5000
MONGO_URI=mongodb://username:password@host:port/database?options
JWT_SECRET=your-jwt-secret-key-here
HF_API_TOKEN=your_hf_api_token_here          # optional, AI features

# Notification provider: 'whatsapp' | 'twilio' | 'alphasms'
SMS_PROVIDER=whatsapp
TWILIO_ACCOUNT_SID=your_account_sid_here
TWILIO_AUTH_TOKEN=your_auth_token_here
TWILIO_FROM_NUMBER=+1234567890
TWILIO_WHATSAPP_FROM=whatsapp:+14155238886

ALPHA_API_KEY=your_alpha_sms_api_key_here
ALPHA_SENDER_ID=
```

### 3. Seed reference data (optional)
```bash
cd server
node seedOffices.js       # seed sample service offices
node scripts/createAdmin.js   # create an initial admin user
```

### 4. Run the app
```bash
# backend (from /server) — http://localhost:5000
npm run dev

# frontend (from /client, separate terminal) — http://localhost:5173
npm run dev
```

## Scripts

| Location | Command | Description |
|---|---|---|
| `server` | `npm run dev` | Start API with hot reload (nodemon) |
| `server` | `npm start` | Start API in production mode |
| `client` | `npm run dev` | Start Vite dev server |
| `client` | `npm run build` | Production build |
| `client` | `npm run lint` | Run ESLint |

## License

Not specified.
