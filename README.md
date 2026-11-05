# Fiora — MVP prototype

A menstrual wellness app: cycle/mood tracking plus a one-tap support alert
to a trusted contact. This is the 6-hour MVP slice — one complete
end-to-end journey, built to be honestly explained in an interview rather
than to look bigger than it is.

## Architecture

```
React (Vite)
   ↓  fetch() + JWT in Authorization header
Node.js + Express REST API
   ↓
MongoDB (via Mongoose)
```

Models live in `backend/models/`: `User`, `Period`, `Mood`,
`FamilyMember`, `Alert`. Each route is a thin controller that calls
straight into Mongoose (`Model.find()`, `Model.create()`, etc.) — no
data-access layer in between.

## What actually works right now

- Register / log in (JWT auth, bcrypt-hashed passwords)
- Dashboard: cycle day + next-period estimate computed from your last
  logged period, plus your most recent mood
- Log a period (start/end date, flow, notes)
- Log a mood (5 options + note)
- Add a trusted contact (name, relationship, phone)
- Send a support alert to a trusted contact — **simulated** unless Twilio
  env vars are set, and the response always says plainly which one
  happened. No SMS is ever silently pretended to have sent.

## What's intentionally not built yet

ML/AI insights, real SMS by default, deployment, automated tests, password
reset, multi-device sync. Say this straight in the interview — it's the
honest next-steps list, not a gap to hide.

## Running it

**Backend** — needs a MongoDB instance. Either:
- Install MongoDB Community Server locally (https://www.mongodb.com/try/download/community) and leave `MONGODB_URI` at its default (`mongodb://127.0.0.1:27017/fiora`), or
- Create a free MongoDB Atlas cluster (https://www.mongodb.com/cloud/atlas) and paste its connection string into `MONGODB_URI`.

```
cd backend
npm install
cp .env.example .env      # then set MONGODB_URI if using Atlas
npm start                 # http://localhost:4000
```

If the connection fails, the server logs the error and exits — check
`MONGODB_URI` first.

**Frontend** (separate terminal)
```
cd frontend
npm install
npm run dev                # http://localhost:5173
```

Open http://localhost:5173 — register an account, log a period and a
mood, add a trusted contact, and hit "Send support alert" to see the
simulated notification.

### Turning on real SMS (optional)

Only if you already have a Twilio account:
```
cd backend
npm install twilio
```
Then fill in `TWILIO_ACCOUNT_SID`, `TWILIO_AUTH_TOKEN`, and
`TWILIO_FROM_NUMBER` in `backend/.env`. With those set, the same
`POST /api/support-alert` route sends a real SMS instead of simulating
one — no frontend changes needed.

## API reference

```
POST /api/auth/register   { name, email, password }
POST /api/auth/login      { email, password }
GET  /api/dashboard                                (auth required)
POST /api/periods         { startDate, endDate?, flow?, notes? }
GET  /api/periods
POST /api/moods           { mood, note? }
GET  /api/moods
POST /api/family          { name, relationship, phone }
GET  /api/family
POST /api/support-alert   { familyMemberId, message? }
GET  /api/support-alert
```
All routes except `/auth/*` require `Authorization: Bearer <token>`.
