# 🏛️ CiviX

CiviX is a civic engagement platform that connects **citizens** with elected/appointed **officials** through petitions, community polls, and direct messaging. Citizens can raise issues and rally support via petitions and polls tied to their locality; officials get a dashboard to respond to petitions, view polls and reports for their area, and track their own activity logs.

## Table of Contents

- [Features](#features)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Environment Variables](#environment-variables)
- [API Overview](#api-overview)
- [Roles & Permissions](#roles--permissions)

## Features

- **Authentication with email OTP verification** — Register as a citizen or official, verify your email with a one-time code, and log in with a JWT-based session. Includes forgot-password / reset-password flows.
- **Official admin verification** — Officials go through an additional `adminVerification` step (pending → requested → verified) tied to an office ID, before gaining official-level access.
- **Petitions** — Citizens can create, edit, delete, sign, and filter petitions by category/location. Petitions carry a status (`active`, `under_review`, `closed`, `resolved`) that officials can update, along with a threaded response.
- **Polls** — Citizens create polls scoped to a state/city with 2+ options, others vote once, and polls auto-expire on a `closesOn` date. Creators (or officials) can close or delete a poll.
- **Officials dashboard** — Officials see petitions/polls filtered to their own locality, respond to petitions with a status update, view aggregate report summaries (total petitions/polls, status breakdown), and pull their own monthly activity logs.
- **Contact / Help & Support** — Citizens can message an official's department directly; officials can view, mark-as-read, and respond to incoming messages.
- **User preferences** — Per-user settings for dark mode, email notifications, in-app alerts, and "critical alerts only," persisted to the user profile.
- **In-app toast notifications** — Lightweight event-based toast system (`notify.js`) for real-time UI feedback (e.g. "Petition signed", "Poll created").

## Tech Stack

**Backend** (`/backend`)
- Node.js + Express 5
- MongoDB + Mongoose
- JSON Web Tokens (`jsonwebtoken`) + `bcryptjs` for auth
- `express-validator` for request validation
- `nodemailer` for OTP / transactional email

**Frontend** (`/frontend`)
- React 19 + Vite
- Axios for API calls
- Chart.js + react-chartjs-2 for the Reports dashboard
- Plain CSS (`civic.css`, `index.css`) — no CSS framework
- Client-side state-based page routing (no react-router; page switching is handled in `App.jsx`, with page state also mirrored to the URL query string)

## Project Structure

```
civix/
├── backend/
│   ├── server.js                 # Express app entry point, route mounting, DB connection
│   ├── config/
│   │   ├── db.js                 # MongoDB connection
│   │   └── emailConfig.js        # Nodemailer/Gmail transport config
│   ├── controllers/
│   │   ├── authController.js     # Register, OTP verify/resend, login, forgot/reset password, profile
│   │   ├── petitionController.js # Create/edit/delete/sign/filter petitions, status updates
│   │   ├── pollController.js     # Create/vote/close/delete polls
│   │   ├── officialController.js # Official petition responses, locality views, report summary, monthly logs
│   │   └── contactController.js  # Citizen <-> official messaging
│   ├── middleware/
│   │   └── authMiddleware.js     # JWT verification middleware
│   ├── models/                   # User, Petition, PetitionResponse, Signature, Poll, ContactMessage, OfficialLog
│   ├── routes/                   # auth, petitions, polls, official, contact, test
│   ├── utils/
│   │   ├── sendEmail.js          # Email sending helper
│   │   └── emailTemplate.js      # OTP / welcome email HTML templates
│   └── .env.example
├── frontend/
│   ├── src/
│   │   ├── login.jsx              # Login/registration UI
│   │   ├── Dashboard.jsx          # Citizen landing dashboard
│   │   ├── Petitions.jsx / CreatePetition.jsx / PetitionDetails.jsx
│   │   ├── Polls.jsx / CreatePoll.jsx
│   │   ├── Officials.jsx          # Official-facing dashboard
│   │   ├── Reports.jsx            # Charts/report summary (Chart.js)
│   │   ├── Settings.jsx / SettingsOfficials.jsx
│   │   ├── HelpSupport.jsx        # Contact/help form
│   │   ├── notify.js              # Toast notification event bus
│   │   ├── App.jsx                # Top-level state, session restore, page switching
│   │   └── civic.css / index.css
│   └── package.json
└── LICENSE
```

## Getting Started

### Prerequisites

- Node.js (v18+) and npm
- MongoDB (local instance or MongoDB Atlas)
- A Gmail account with an [App Password](https://support.google.com/accounts/answer/185833) (for sending OTP emails), unless running in dev mode

### 1. Clone & install

```bash
git clone <repo-url>
cd civix

# Backend
cd backend
npm install

# Frontend
cd ../frontend
npm install
```

### 2. Configure environment variables

```bash
cd backend
cp .env.example .env
```

Then edit `.env` with your own values (see [Environment Variables](#environment-variables)).

> **Tip:** Set `OTP_DEV_MODE=true` while developing locally so OTP codes print to the server console instead of requiring real email delivery.

### 3. Run MongoDB

Make sure MongoDB is running locally, or point `MONGO_URI` at an Atlas cluster.

### 4. Start the backend

```bash
cd backend
npm run dev
```

The API runs on `http://localhost:5000` by default.

### 5. Start the frontend

```bash
cd frontend
npm run dev
```

Vite will serve the app (default `http://localhost:5173`). Note the frontend currently points at `http://localhost:5000/api/...` directly in a few places (e.g. `App.jsx`), so update those if your backend runs elsewhere.

## Environment Variables

Defined in `backend/.env` (see `backend/.env.example`):

| Variable | Description | Example |
|---|---|---|
| `PORT` | Backend server port | `5000` |
| `MONGO_URI` | MongoDB connection string | `mongodb://127.0.0.1:27017/civiv` |
| `JWT_SECRET` | Secret used to sign JWTs | a long random string |
| `EMAIL_USER` | Gmail address used to send OTP emails | `your-email@gmail.com` |
| `EMAIL_PASS` | Gmail App Password (16 chars) — not your normal password | `xxxxxxxxxxxxxxxx` |
| `OTP_DEV_MODE` | `true` logs OTP to console (dev only); `false` sends via email | `false` |

**Security notes:** never commit `.env`, and never share real email/app-password credentials. Set the same variables in your hosting provider's environment settings for production.

## API Overview

All routes are mounted under `/api` in `backend/server.js`:

| Base path | Purpose |
|---|---|
| `/api/auth` | Register, verify-email (OTP), resend OTP, login, forgot/reset password, get/update profile (`/me`), change password |
| `/api/petitions` | Create, list all, list by official's locality, edit, delete, filter, update status, sign, official response |
| `/api/polls` | Create, list all, list by official's locality, vote, close, delete |
| `/api/official` | Official petition response, official's petitions, report summary, monthly activity logs |
| `/api/contact` | Send message, citizen's own messages, official inbox, respond, mark as read |
| `/api/test` | Simple health-check route |

Most write endpoints and any locality/role-scoped reads require a `Bearer` JWT (via `authMiddleware`).

## Roles & Permissions

CiviX has three user roles (`backend/models/User.js`):

- **citizen** — default role; can create/sign petitions, create/vote on polls, and message officials.
- **official** — additionally sees petitions/polls scoped to their locality, can respond to and update petition status, view report summaries, and view their own monthly action logs. Requires `adminVerification` (office ID) to be approved.
- **admin** — highest privilege role (not self-assignable at registration — only `citizen` and `official` can be selected on sign-up).

---

*This README was generated from a review of the project's backend and frontend source. If any setup step doesn't match your local environment, check `backend/README.md` and `frontend/README.md` for additional notes.*
