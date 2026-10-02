# Deploy the standalone project

## A. PostgreSQL (separate)

Use a PostgreSQL provider or your own server. Create a database and copy its connection string.
Do NOT use localhost in a hosted backend database URL.
The provider's connection string may include `sslmode=require`; keep it.

## B. Node.js backend (separate)

Deploy this repository on a Node hosting provider. The app runs as a long-lived Express process.
Use the repository root:

- Install/build command: `npm ci`
- One-time database setup command: `npm run db:setup`
- Start command: `npm start`

Set backend environment variables:

- NODE_ENV=production
- DATABASE_URL=your private PostgreSQL URL
- FRONTEND_ORIGINS=https://YOUR-SITE.netlify.app (or your exact custom domain; comma-separated if needed)
- ADMIN_EMAIL=your administrator email
- ADMIN_PASSWORD=a unique password of at least 12 characters
- SEED_DEMO_ACCOUNTS=true for the sample project, otherwise false
- DEMO_PASSWORD=your chosen demo password
- DEMO_PIN=your chosen six-digit demo PIN
- PAYMENT_MODE=demo or razorpay-test
- PRINT_DEMO_CREDENTIALS=false
- PORT is normally provided by the backend host

Set admin credentials before the first database setup. Production never prints credentials to logs.
Do not set a public client variable for DATABASE_URL or any secrets.
Get the HTTPS backend origin, e.g. https://YOUR-BACKEND.example.com.
Visit its /api/health endpoint to check database connectivity.

Documents are stored in PostgreSQL as private bytes; no public upload folder or storage volume is required for this version. A dedicated encrypted object store would be a future upgrade for substantial document volumes.
The demo is not an automated KYC service; admin document reviews are manual.

## C. React frontend on Netlify

Upload this project to YOUR Git repository, without node_modules or server/.env, and import it in Netlify.
Netlify picks up the included netlify.toml:

- Base directory: repository root (leave blank)
- Build command: npm run build
- Publish directory: client/dist
- Node version: 22

Set the Netlify BUILD environment variable:

```text
NETLIFY_BACKEND_URL=https://YOUR-BACKEND.example.com
```

Use only the HTTPS origin, without `/api` or any path. Redeploy after changing it.
The build generates these rules in client/dist/_redirects:

```text
/api/* https://YOUR-BACKEND.example.com/api/:splat 200
/* /index.html 200
```

The React client always requests relative `/api/...` URLs. Netlify proxies them to the separate backend so session cookies remain attached to the frontend origin. Keep the backend FRONTEND_ORIGINS aligned with your deployed frontend URL.
Use HTTPS in production. Production cookies are HttpOnly, Secure and SameSite=Lax.
For changing backend origins, regenerate the frontend build and redeploy.

### Manual Netlify upload

After deploying the backend, from the ROOT project in PowerShell:

```powershell
$env:NETLIFY_BACKEND_URL="https://YOUR-BACKEND.example.com"
npm.cmd run build
```

Upload the contents of `client/dist` to Netlify. Uploading source folders or the full ZIP is not a frontend deployment.
The backend and database still need to be deployed separately.

## Scope

Netlify compatibility has been implemented and the static build/proxy generation tested. This ZIP has not been deployed to your Netlify/backend/database accounts.
Wallet top-ups are demo or Razorpay TEST. Numbers and SMS are simulated until a real virtual-number provider is integrated. The included gateway rejects LIVE Razorpay keys.
Passwords and PINs are hashed. Every customer API query is scoped to the authenticated account. Admin-only actions require the administrator role; frontend state does not grant permissions.
