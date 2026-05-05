# 🌿 MindBloom — Backend Setup Guide

## What's included

```
mindbloom-backend/
├── server.js               ← Express entry point
├── package.json
├── .env.example            ← Copy to .env and edit
├── db/
│   └── database.js         ← SQLite schema + init
├── middleware/
│   └── auth.js             ← JWT verification middleware
├── routes/
│   ├── auth.js             ← Register / Login / Logout / /me
│   ├── users.js            ← Profile update, password change, login history
│   ├── checkins.js         ← Daily mood check-ins + heatmap endpoint
│   ├── journals.js         ← Journal CRUD
│   └── chats.js            ← Chat history persistence
└── public/
    └── mindbloom-api.js    ← Drop this into your frontend HTML
```

A `mindbloom.db` SQLite file is auto-created on first run — no external DB needed.

---


### 1. Prerequisites
- **Node.js 18+** — download from https://nodejs.org

### 2. Install & run

```bash
# 1. Place your digital-therapist.html in the public/ folder
cp /path/to/digital-therapist.html public/

# 2. Install dependencies
npm install

# 3. Create .env file
cp .env.example .env
# Open .env and set a strong JWT_SECRET (any long random string)

# 4. Start the server
npm start
# → 🌿 MindBloom server running on http://localhost:3000
```

Open **http://localhost:3000** in your browser — you'll see the app.

### 3. Wire the frontend to the backend

Add **one script tag** at the very end of `digital-therapist.html`, just before `</body>`:

```html
<script src="mindbloom-api.js"></script>
```

That's it. The `mindbloom-api.js` file:
- **Overrides** `doLogin()`, `doSignup()`, `doLogout()` with real API calls
- **Auto-restores** the session on page reload (JWT stored in localStorage)
- **Saves** check-ins, journal entries, and chat messages to the database
- **Loads** real heatmap data from the database on login

---

## API Reference

All protected endpoints require the header:
```
Authorization: Bearer <token>
```
The token is returned on login/register and auto-attached by `mindbloom-api.js`.

### Auth

| Method | Path | Body | Description |
|--------|------|------|-------------|
| POST | `/api/auth/register` | `{name, email, password}` | Create account → returns `{token, user}` |
| POST | `/api/auth/login`    | `{email, password}`       | Sign in → returns `{token, user}` |
| POST | `/api/auth/logout`   | —                         | Server-side logout log |
| GET  | `/api/auth/me`       | —                         | Validate token, fetch profile |

### Users (all protected)

| Method | Path | Body | Description |
|--------|------|------|-------------|
| GET    | `/api/users/me`           | — | Get profile |
| PUT    | `/api/users/me`           | `{name, age, phone, pronouns, therapist, dark_mode}` | Update profile |
| PUT    | `/api/users/me/password`  | `{current_password, new_password}` | Change password |
| GET    | `/api/users/me/logins`    | — | Login history (last 50) |
| DELETE | `/api/users/me`           | `{password}` | Delete account |

### Check-ins (all protected)

| Method | Path | Body | Description |
|--------|------|------|-------------|
| POST   | `/api/checkins`         | `{mood(1–5), energy, anxiety, sleep, note, tags}` | Save check-in |
| GET    | `/api/checkins`         | `?limit&offset` | List check-ins |
| GET    | `/api/checkins/heatmap` | — | 90-day mood array |
| GET    | `/api/checkins/:id`     | — | Single check-in |
| DELETE | `/api/checkins/:id`     | — | Delete check-in |

### Journals (all protected)

| Method | Path | Body | Description |
|--------|------|------|-------------|
| POST   | `/api/journals`     | `{title, body, mood_tag}` | Create entry |
| GET    | `/api/journals`     | `?limit&offset` | List entries |
| GET    | `/api/journals/:id` | — | Full entry |
| PUT    | `/api/journals/:id` | `{title, body, mood_tag}` | Update entry |
| DELETE | `/api/journals/:id` | — | Delete entry |

### Chat (all protected)

| Method | Path | Body | Description |
|--------|------|------|-------------|
| POST   | `/api/chats` | `[{role, content, risk_level}]` | Save messages |
| GET    | `/api/chats` | `?limit` | Conversation history |
| DELETE | `/api/chats` | — | Clear history |

---

## Database tables

| Table | Purpose |
|-------|---------|
| `users` | Accounts — name, email, hashed password, profile fields |
| `login_log` | Every login event with timestamp and IP |
| `checkins` | Daily mood/energy/anxiety/sleep scores |
| `journals` | Journal entries (title + body) |
| `chat_messages` | Full conversation history with risk level |

Passwords are hashed with **bcrypt (cost 12)** — never stored in plain text.

---

## Security notes

- Always change `JWT_SECRET` in `.env` before deploying
- Use HTTPS in production (add a reverse proxy like Nginx or use Railway/Render)
- Set `FRONTEND_URL` in `.env` to your actual domain to restrict CORS
- For production, consider rate-limiting the `/api/auth/login` route

---

## Development mode (auto-reload)

```bash
npm run dev   # uses nodemon — restarts on file save
```

---

## Deploying to the cloud (free options)

### Railway
```bash
npm install -g @railway/cli
railway login
railway init
railway up
```

### Render
- Push to GitHub, create a new "Web Service" on render.com
- Build command: `npm install`
- Start command: `node server.js`
- Add environment variables in the Render dashboard

---

## Troubleshooting

**`Error: Cannot find module 'better-sqlite3'`**  
→ Run `npm install` again. If it fails on Windows, install the Visual Studio C++ build tools.

**`SQLITE_CONSTRAINT_UNIQUE`**  
→ The email is already registered. Use Sign In instead.

**401 Unauthorized on all requests**  
→ Token expired. Sign out and sign back in.

**CORS error in browser console**  
→ Make sure the backend is running on port 3000, and `API_BASE` in `mindbloom-api.js` matches.
