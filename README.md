# Activity Tracker (Android + Web)

A consent-based activity-sharing app prototype. It is **not a WhatsApp tracker** and does not access WhatsApp Last Seen, online presence, messages, or private account data. Activity timestamps are created only when a participant checks in inside this app.

## Features
- Phone-number + password registration and sign-in
- Password hashing (bcrypt) and signed sessions (JWT)
- Invite registered participants by phone number
- Incoming invitation acceptance or decline
- Sharing opt-in, pause, and connection revocation
- Contacts dashboard showing in-app activity check-ins
- Responsive web interface and Capacitor Android wrapper
- SQLite database for local development

## Requirements
- Node.js 20 or newer and npm
- Android Studio only if you want to build/install the Android APK

## Run locally

1. Copy `server/.env.example` to `server/.env` (or create the file using the example below).
2. Set a strong random `JWT_SECRET` (at least 32 characters).
3. Install dependencies from the project root and each package:

```bash
npm install
npm install --prefix server
npm install --prefix client
```

4. Start API and web app:

```bash
npm run dev
```

Open `http://localhost:5173`. API runs at `http://localhost:4000`.

Example `server/.env`:

```env
PORT=4000
JWT_SECRET=replace-with-a-long-random-secret-at-least-32-characters
CLIENT_ORIGIN=http://localhost:5173
```

For a deployed web frontend, set `CLIENT_ORIGIN` to its exact HTTPS origin. Set `VITE_API_URL` in `client/.env` to the deployed API URL plus `/api` before building.

## Test the consent flow
1. Register two accounts using different phone numbers in international format (e.g. `+923001234567`).
2. Sign in as account A and send an invitation to account B's registered number.
3. Sign in as account B and accept the invitation under **Invitations**.
4. Enable **Share my activity** in account B and press **I'm active**.
5. Sign in as account A. The accepted contact can now see account B's in-app check-in time.
6. Account B can pause sharing at any time. Either participant can revoke the connection.

## Android build

From the project root:

```bash
npm install
npm install --prefix client
npm install --prefix server
npm run build
npx cap add android
npx cap sync android
npx cap open android
```

In Android Studio, select Build > Build Bundle(s) / APK(s) > Build APK(s). The generated Android project is intentionally not committed; Capacitor creates it locally.

## API overview
- `POST /api/auth/register` — create account
- `POST /api/auth/login` — sign in
- `GET /api/me` — current profile
- `PATCH /api/me/sharing` — enable/disable sharing
- `POST /api/activity/ping` — record an in-app activity check-in
- `POST /api/invitations` — invite a registered phone number
- `GET /api/invitations` — incoming/outgoing invitations
- `POST /api/invitations/:id/respond` — accept/decline
- `DELETE /api/invitations/:id` — revoke a connection
- `GET /api/contacts` — accepted contacts
- `GET /api/activity/:userId` — read activity with an accepted connection

## Before public launch — important
This is a development starter, not a production-audited service. Before real users use it:
- Add phone ownership verification (SMS OTP) through a provider; currently a phone number is an account identifier, not verified ownership.
- Use HTTPS everywhere and set strong production secrets; never commit `.env` or the SQLite database.
- Add account recovery, data export/deletion, abuse reporting, privacy policy, terms, and retention controls.
- Consider PostgreSQL for deployment, database backups, monitoring, and stricter rate limits.
- Add CSRF protections if switching to cookie-based auth, and review CORS for your exact domains.
- Add tests, security review, and store-policy review before release.
- Do not add scraping, WhatsApp Web automation, unofficial clients, or bypasses for WhatsApp privacy settings.

## Architecture
`client/` is a React + Vite responsive app. `server/` is an Express API with SQLite. Capacitor wraps the same built web app for Android. No WhatsApp API integration is included because WhatsApp does not expose another person's Last Seen/online status through a public API.
