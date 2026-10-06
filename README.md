# Student_management — School Management System

A full-stack school management app: a public site with an admission form, a student dashboard (teachers, notices, calendar, discussion, results, fees, profile) and an admin panel (admissions, students, teachers, fees, notices, calendar, anonymous posts).

## Tech Stack

| Layer | Technology |
|---|---|
| Backend | Node.js, Express 4, MySQL (`mysql2`), `express-session` + `express-mysql-session`, `helmet`, `cors`, `express-rate-limit`, `express-validator`, `multer`, `bcryptjs`, `dotenv` |
| Frontend | React 18, Vite 7, React Router 7, Tailwind CSS 4, Axios, framer-motion, lucide-react, sweetalert2 |
| Database | MySQL 8 — schema `DataBase/school_management.sql`, sample data `DataBase/seed_data.sql` |

## Project Structure

```
Student_management/
├── Back_end/                 # Express REST API
│   ├── app.js                # Entry point, mounts all routes
│   ├── config/               # db.js, session.js
│   ├── controllers/          # Route handlers
│   ├── middleware/           # auth, validate, rateLimiters, upload, errorHandler
│   ├── models/               # SQL queries
│   ├── routes/               # /api/* routers
│   ├── validators/           # express-validator chains
│   ├── scripts/seed.js       # Loads sample data
│   ├── seedAdmin.js          # Creates the admin account
│   └── .env.example          # Env template (copy to .env)
├── Front_end/                # React + Vite SPA
│   └── src/
│       ├── main.jsx          # Router + auth providers
│       ├── components/
│       │   ├── Admin/        # Admin panel pages + AdminLayout
│       │   ├── Auth/         # Landing, login, admission, ProtectedRoute
│       │   ├── Teacher/ Notice/ Calendar/ Discuss/ Fees/ Results/ Profile/
│       │   ├── Home/         # Navbar
│       │   └── Server/       # Axios instance + API services
│       └── index.css         # Tailwind v4 theme tokens
└── DataBase/
    ├── school_management.sql # Schema
    └── seed_data.sql         # Sample data
```

## Prerequisites

- Node.js 18+ (tested on v24)
- MySQL 8 running locally (or a reachable host)

## Getting Started

### 1. Database

```bash
mysql -u root -p < DataBase/school_management.sql
```

### 2. Backend

```bash
cd Back_end
npm install
cp .env.example .env      # Windows: copy .env.example .env
# edit .env -> set DB_PASSWORD, keep the generated SESSION_SECRET
node scripts/seed.js      # optional: load sample data
node seedAdmin.js         # create the admin account
npm run dev               # nodemon on http://localhost:5000
```

### 3. Frontend

```bash
cd Front_end
npm install
cp .env.example .env      # already set for local dev
npm run dev               # Vite on http://localhost:5173
```

Open `http://localhost:5173`.

## Environment Variables

### `Back_end/.env`

| Key | Description | Default |
|---|---|---|
| `NODE_ENV` | `development` / `production` | `development` |
| `PORT` | API port | `5000` |
| `DB_HOST`, `DB_PORT` | MySQL host / port | `localhost`, `3306` |
| `DB_USER`, `DB_PASSWORD` | MySQL credentials | `root`, *(empty)* |
| `DB_NAME` | Database name | `school_management` |
| `DB_CONNECTION_LIMIT` | Pool size | `10` |
| `SESSION_SECRET` | Cookie signing secret — **never commit** | generated |
| `SESSION_COOKIE_NAME` | Session cookie name | `school_session` |
| `SESSION_MAX_AGE_MS` | Session lifetime in ms | `86400000` |
| `CORS_ORIGIN` | Allowed origins, comma-separated | `http://localhost:3000,http://localhost:5173` |
| `UPLOAD_DIR` | Uploads folder | `uploads` |
| `ADMIN_ID`, `ADMIN_PASSWORD` | Used by `seedAdmin.js` | — |

### `Front_end/.env`

| Key | Description |
|---|---|
| `VITE_API_URL` | API base URL (`http://localhost:5000/api`) |
| `VITE_UPLOADS_URL` | Base URL for uploaded files (`http://localhost:5000`) |

`.env` files are git-ignored; only `.env.example` is committed.

## API Reference

Base URL: `http://localhost:5000/api` — health check: `GET /api/health`

| Group | Endpoint | Auth | Notes |
|---|---|---|---|
| Auth | `POST /auth/admin/login`, `POST /auth/student/login` | — | Rate-limited, sets httpOnly session cookie |
| | `POST /auth/logout`, `GET /auth/me` | session | |
| Admission | `POST /admission/apply` | — | Public application form (multipart, file uploads) |
| Teachers | `GET /teachers`, `GET /teachers/:id` | — | Public directory |
| Notices | `GET /notices`, `GET /notices/:id` | — | Public |
| Calendar | `GET /calendar`, `GET /calendar/:id` | — | Public |
| Posts | `GET /posts`, `POST /posts`, `POST /posts/:id/upvote`, `POST /posts/:id/downvote`, `GET /posts/:id/comments`, `POST /posts/:id/comments` | student | Anonymous board, voting rate-limited |
| Student | `GET /students/me`, `PATCH /students/me`, `GET /students/me/fees`, `GET /students/me/results`, `POST /students/me/change-password` | student | |
| Admin | `/admin/admissions`, `/admin/students`, `/admin/teachers`, `/admin/notices`, `/admin/calendar`, `/admin/posts`, `/admin/fees` | admin | CRUD + admission status updates, fee assignment & payment marking |

Route files live in `Back_end/routes/`; all handlers are wrapped in `asyncHandler` and validated by `middleware/validate.js`.

## Frontend Routes

| Path | Access | Page |
|---|---|---|
| `/` | public | Landing |
| `/login` | public | Student login |
| `/admission` | public | Admission form |
| `/dashboard/*` | student | `teachers`, `notices`, `calendar`, `posts`, `results`, `fees`, `profile` |
| `/admin/login` | public | Admin login |
| `/admin/*` | admin | `admissions`, `students`, `teachers`, `notices`, `calendar`, `posts`, `fees` |

Guards: `Auth/ProtectedRoute.jsx` for students, `Admin/AdminLayout.jsx` for admin. Both contexts restore the session via `GET /auth/me` on mount and clear state on any `401` (`Server/api.js` interceptor).

## Scripts

| Location | Command | Purpose |
|---|---|---|
| `Back_end` | `npm run dev` | API with nodemon |
| `Back_end` | `npm start` | API (production) |
| `Back_end` | `node scripts/seed.js` | Insert sample data |
| `Back_end` | `node seedAdmin.js` | Create/update admin from `.env` |
| `Front_end` | `npm run dev` | Vite dev server |
| `Front_end` | `npm run build` | Production build → `dist/` |
| `Front_end` | `npm run lint` | ESLint |
| `Front_end` | `npm run preview` | Preview the build |

## Security Notes

- Sessions are server-side (MySQL-backed) with httpOnly cookies; `secure` is enabled when `NODE_ENV=production`.
- Passwords stored as bcrypt hashes; login endpoints are rate-limited.
- `helmet` headers, CORS restricted to `CORS_ORIGIN`, JSON body capped at 1 MB.
- Never commit `.env` — rotate `SESSION_SECRET` and `ADMIN_PASSWORD` for production.

## License

MIT
