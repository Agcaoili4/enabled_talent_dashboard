# Deployment Guide: Vercel Frontend + Render Backend + Render Postgres

## Overview

This project is deployed as:

- Frontend on **Vercel**
- Backend API (Express/Node) on **Render Web Service**
- Database on **Render PostgreSQL**

This guide documents the exact deployment flow completed for this project.

---

## 1. Create Render Services

### 1.1 Create Render account

- Sign in to Render.
- Connect GitHub account.

### 1.2 Create Backend Web Service

- In Render, create a **New Web Service** from the GitHub repo.
- Configure:
  - **Root Directory**: `enabled-talent-dashboard/Backend`
  - **Build Command**: `npm install --include=dev && npm run build`
  - **Start Command**: `npm start`

Notes:

- Build command includes dev dependencies so TypeScript types are available at build time.
- `npm start` runs `node dist/server.js`.

---

## 2. Configure Backend Environment Variables (Render)

Set these in the Render Web Service:

- `NODE_ENV=production`
- `DATABASE_URL=<Render Postgres connection string>`
- `ALLOWED_ORIGINS=<comma-separated frontend domains>`

Example:
`ALLOWED_ORIGINS=https://your-app.vercel.app,https://your-custom-domain.com`

Important:

- Do not commit `.env` files containing production secrets.
- Keep secrets only in Render/Vercel environment variable settings.

---

## 3. Create Render PostgreSQL Database

- In Render, create a **PostgreSQL** service.
- Copy the DB connection details.
- Attach the correct DB URL to the backend service via `DATABASE_URL`.

---

## 4. Import Database Schema

Two supported methods:

### Method A: pgAdmin (used in this setup)

- Add a new server in pgAdmin using Render DB connection details.
- Do not paste the full URL into the Host field.
- Use separate fields:
  - Host
  - Port
  - Database
  - Username
  - Password
- Set SSL mode to `require`.
- Open Query Tool and execute the SQL schema script (`schema.sql`) to create tables.

### Method B: psql (optional)

```bash
psql "YOUR_RENDER_EXTERNAL_DATABASE_URL?sslmode=require" -f schema.sql
```

---

## 5. Connect Frontend (Vercel) to Backend (Render)

In Vercel project settings, add:

- `VITE_API_URL=https://<your-render-service>.onrender.com`

Then redeploy Vercel.

Frontend requests will use this backend URL in production.

---

## 6. Verify Deployment

### Backend health check

Open:
`https://<your-render-service>.onrender.com/api/health`

Expected:

- `{ "ok": true, ... }`

### Frontend check

- Open Vercel frontend URL.
- Confirm dashboard loads and API-driven data appears.

---

## 7. Troubleshooting Notes

### Issue: `Cannot find module dist/server.js`

Cause:

- TypeScript output path mismatch.

Fix:

- Ensure backend `tsconfig.json` compiles to `dist` (`rootDir: src`, `outDir: dist`).
- Keep start command: `node dist/server.js`.

### Issue: TypeScript missing declarations (`@types/*`)

Cause:

- Dev dependencies not installed during build.

Fix:

- Build command must include dev deps:
  `npm install --include=dev && npm run build`

### Issue: DB health returns `ok: false`

Cause:

- Incorrect DB URL, SSL mismatch, or wrong environment values.

Fix:

- Confirm `DATABASE_URL` is correct for Render Postgres.
- Confirm SSL requirements are satisfied.
- Confirm backend has been redeployed after env var updates.

### Issue: CORS errors

Cause:

- Frontend domain not included in `ALLOWED_ORIGINS`.

Fix:

- Add all required Vercel domains (production + custom domain) to `ALLOWED_ORIGINS`.

---

## 8. Handoff Checklist

- Render Web Service is live.
- Render PostgreSQL is live.
- Schema created in Render DB.
- Backend env vars configured in Render.
- Frontend env var (`VITE_API_URL`) configured in Vercel.
- Frontend redeployed successfully.
- Health endpoint returns success.

---

## Security Best Practices

- Never share plaintext credentials in chat, email, or docs.
- Rotate any secret if it was accidentally exposed.
- Restrict DB/network access where possible.
- Use least-privilege credentials for production environments.
