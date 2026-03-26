# Deployment Guidelines: Vercel + Render + Render Postgres

## Purpose
This document defines the standard deployment process for the Enabled Talent Dashboard.

## Architecture Standard
- Frontend: **Vercel**
- Backend API: **Render Web Service** (Node/Express)
- Database: **Render PostgreSQL**

## Prerequisites
- GitHub repository connected to Vercel and Render
- Render Web Service and PostgreSQL service created
- Production domains confirmed

## Backend Deployment Guidelines (Render)
1. Create or open Render Web Service.
2. Configure:
   - `Root Directory`: `enabled-talent-dashboard/Backend`
   - `Build Command`: `npm install --include=dev && npm run build`
   - `Start Command`: `npm start`
3. Required environment variables:
   - `DATABASE_URL=<render postgres url>`
   - `NODE_ENV=production`
   - `ALLOWED_ORIGINS=<comma-separated allowed origins>`
4. Redeploy after any environment-variable change.

## CORS Guideline (Critical)
- The backend expects `ALLOWED_ORIGINS` (not `CORS_ORIGIN`).
- Include all active frontend domains.

Example:
`ALLOWED_ORIGINS=http://localhost:5173,https://enabledtalentdashboard.vercel.app,https://enabledtalentdashboard-p076jw6ku-agcaoili4s-projects.vercel.app`

## Frontend Deployment Guidelines (Vercel)
1. Set environment variable:
   - `VITE_API_URL=https://enabled-talent-dashboard.onrender.com`
2. Redeploy Vercel after env updates.
3. Confirm deployed branch is `main` (or agreed production branch).

## Database Guidelines
1. Connect to Render PostgreSQL (pgAdmin or `psql`).
2. Apply schema (`schema.sql`).
3. Apply seed/test data as needed.
4. Validate record counts for key tables (`Students`, `Applications`, `Placements`).

## Validation Checklist
1. Backend health endpoint returns success:
   - `https://enabled-talent-dashboard.onrender.com/api/health`
2. Dashboard API returns data:
   - `https://enabled-talent-dashboard.onrender.com/api/dashboard`
3. Frontend Network tab shows calls to:
   - `https://enabled-talent-dashboard.onrender.com/api/...`
4. Dashboard cards/charts reflect live DB changes.

## Troubleshooting Guidelines
### CORS blocked (`Origin ... is not allowed`)
- Confirm `ALLOWED_ORIGINS` contains exact frontend URL(s).
- Redeploy backend.

### `Cannot find module dist/server.js`
- Ensure backend TypeScript output is `dist/`.
- Confirm start command remains `npm start`.

### Frontend stays at zeros
- Confirm frontend is calling `VITE_API_URL`, not localhost.
- Confirm API responses are `200`.
- Confirm database actually contains records.

## Security Guidelines
- Never commit secrets to Git.
- Keep credentials only in platform environment variables.
- Rotate credentials if exposed.
