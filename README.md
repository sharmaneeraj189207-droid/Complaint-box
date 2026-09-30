# Smart Automated Complaint Management System (SmartResolve AI)

An end-to-end, enterprise-grade automated complaint management and resolution platform adhering strictly to the **Smart Automation Challenge specification**. Built with a modern decoupled architecture featuring a React (Vite) frontend, Node.js/Express backend, MongoDB with Mongoose, and the **Google Gemini API** (`@google/genai`) configured in strict JSON schema response mode with an intelligent fallback heuristic engine.

---

## 🌟 Key Architecture & Highlights

- **Decoupled Monorepo Structure**: Isolated `/client` and `/server` packages orchestrated with root workspace scripts.
- **Strict Server-Side AI Execution**: The Gemini API key resides solely in `server/.env` (`process.env.GEMINI_API_KEY`) and is never leaked to the client.
- **Strict JSON Schema Mode**: Utilizes `@google/genai` with `responseSchema` to guarantee structured extraction:
  - `category`: `["Billing", "Technical", "Facilities", "HR", "Academic", "General"]`
  - `priority`: `["Low", "Medium", "High", "Critical"]`
  - `sentiment`: `["Positive", "Neutral", "Frustrated", "Urgent"]`
  - `summary`: 1-2 sentence executive overview
  - `suggestedAction`: Immediate actionable remediation step
- **Fault-Tolerant Fallback Triage**: If API quota limits or network outages occur, an embedded heuristic rule engine classifies the ticket so grievances are **never dropped**.
- **Automated Department Routing**: Auto-assigns complaints based on AI categorization to departments such as *IT & Technical Support*, *Billing & Finance*, *Facilities & Maintenance*, etc.
- **Visual 5-Stage Status Timeline**: Real-time tracking from `Submitted` ➔ `Assigned` ➔ `In Progress` ➔ `Resolved` ➔ `Closed`.
- **Role-Based Access Control (RBAC)**: Distinguishes between regular submitters and staff administrators using JWT Bearer authentication and bcryptjs password encryption.
- **Bi-directional Zod Validation**: Strict schema enforcement on both frontend forms and backend API routes.

---

## 📂 Project Directory Layout

```text
complaint-system/
├── package.json                   # Root orchestrator scripts (concurrently dev runner)
├── README.md                      # Architecture documentation & API guide
├── server/
│   ├── .env                       # Active server environment variables
│   ├── .env.example               # Environment variables template
│   ├── package.json               # Node.js backend configuration (ES Modules)
│   ├── server.js                  # Express bootstrap, CORS, and routing mount
│   ├── seed.js                    # Database seeder with demo accounts & tickets
│   ├── config/
│   │   └── db.js                  # Mongoose MongoDB connection manager
│   ├── models/
│   │   ├── User.js                # User model with bcrypt pre-save hashing & roles
│   │   └── Complaint.js           # Complaint schema with AI metadata & audit timeline
│   ├── middleware/
│   │   ├── auth.js                # JWT Bearer authentication & admin guard
│   │   └── validate.js            # Zod validation middleware & schemas
│   ├── services/
│   │   └── geminiService.js       # @google/genai integration + heuristic fallback engine
│   ├── controllers/
│   │   ├── authController.js      # Register, login, and profile fetching
│   │   └── complaintController.js # Intake, tracking, metrics, and status transition
│   └── routes/
│       ├── authRoutes.js          # /api/auth endpoints
│       └── complaintRoutes.js      # /api/complaints endpoints
└── client/
    ├── .env                       # Active client environment variables
    ├── .env.example               # Client environment template
    ├── package.json               # Vite React client dependencies
    ├── vite.config.js             # Vite config with backend /api proxy
    ├── tailwind.config.js         # Tailwind CSS styling and color palette
    ├── postcss.config.js          # PostCSS configuration
    ├── index.html                 # HTML5 template with Plus Jakarta Sans typography
    └── src/
        ├── main.jsx               # React DOM entry point
        ├── App.jsx                # React Router v6 navigation and layout
        ├── index.css              # Tailwind base, utilities, and glassmorphism
        ├── api/
        │   └── axiosInstance.js   # Centralized Axios client with JWT interceptor
        ├── context/
        │   └── AuthContext.jsx    # Authentication & user session context
        ├── components/
        │   ├── Navbar.jsx         # Responsive navigation bar with role badge
        │   ├── ProtectedRoute.jsx # Role-based route guard
        │   ├── StatusBadge.jsx    # Color-coded workflow stage badge
        │   └── PriorityBadge.jsx  # Severity indicator with pulse animations
        └── pages/
            ├── Login.jsx          # Login with one-click demo autofill
            ├── Register.jsx       # Registration with role selection
            ├── SubmitComplaint.jsx# Grievance intake with live AI preview & templates
            ├── TrackComplaint.jsx # Visual 5-stage status timeline & ticket lookup
            ├── UserDashboard.jsx  # Submitter portal with past history & metrics
            └── AdminDashboard.jsx # Staff operations console with metrics & drawer
```

---

## ⚡ Quick Start Guide

### 1. Prerequisites
- **Node.js**: v18.0.0 or higher (v20.x recommended)
- **MongoDB**: A running local MongoDB daemon (`mongodb://127.0.0.1:27017`) or a free [MongoDB Atlas](https://www.mongodb.com/cloud/atlas) connection URI.

### 2. Configure Environment Variables
Verify or edit `server/.env`:
```env
PORT=5000
MONGODB_URI=mongodb://127.0.0.1:27017/complaint_system
JWT_SECRET=production_grade_jwt_secret_complaint_box_ai_2025
JWT_EXPIRES_IN=7d
GEMINI_API_KEY=your_gemini_api_key_here
CLIENT_URL=http://localhost:5173
```
> **Note**: If you don't have a Gemini API key yet, the application automatically engages its intelligent keyword and sentiment heuristic engine without crashing or dropping complaints!

### 3. Seed Database with Demo Accounts & Tickets (Optional)
Run the automated seed script to populate sample accounts and grievances:
```bash
npm run seed
```

Default credentials created:
| Role | Email | Password |
|---|---|---|
| **Staff Admin** | `admin@complaint.io` | `Admin@123` |
| **Standard User** | `user@complaint.io` | `User@123` |

### 4. Run Both Server and Client Concurrently
From the root workspace directory:
```bash
npm run dev
```

Or run each service individually:
- **Backend API**: `npm run server` (runs at `http://localhost:5000`)
- **Frontend Client**: `npm run client` (runs at `http://localhost:5173`)

---

## 🔌 API Reference Summary

### Authentication Endpoints (`/api/auth`)
| Method | Route | Description | Auth Required |
|---|---|---|---|
| `POST` | `/api/auth/register` | Register new user or administrator | Public |
| `POST` | `/api/auth/login` | Authenticate user and issue JWT Bearer token | Public |
| `GET` | `/api/auth/me` | Fetch active user profile from session | User / Admin |

### Complaint Endpoints (`/api/complaints`)
| Method | Route | Description | Auth Required |
|---|---|---|---|
| `POST` | `/api/complaints` | Ingest complaint, run Gemini AI triage, auto-route | Public / User |
| `GET` | `/api/complaints/track/:trackingId` | Query ticket details and visual timeline | Public |
| `GET` | `/api/complaints/my` | Retrieve all grievances submitted by current user | User |
| `GET` | `/api/complaints/metrics` | Retrieve operational metrics & category breakdowns | Admin Only |
| `GET` | `/api/complaints` | Query complaints with status/priority/dept filters | Admin Only |
| `PATCH` | `/api/complaints/:id/status` | Transition status and add audit/resolution notes | Admin Only |
#   C o m p l a i n t - b o x  
 