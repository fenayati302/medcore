# MedCore.tech deployment

Two frontends on one domain family. **Backend stays at `https://api.hearthmeet.com`** (no API host change).

| URL | App | Repo path |
|-----|-----|-----------|
| `https://medcore.tech` | MedCore landing | `nodemon-site/medcore/` |
| `https://www.medcore.tech` | → redirect to apex | Amplify domain redirect |
| `https://sage.medcore.tech` | Sage — AI Receptionist | `sage-realtor-app/dist/` |
| `https://melia.medcore.tech` | Melia — GC AI Receptionist | `forge-contractor-app/dist/` |

## 1. DNS (Route 53 or registrar)

Point both hostnames at AWS Amplify after connecting the domain in each app:

- `medcore.tech` → Amplify app **medcore-landing**
- `www.medcore.tech` → CNAME to Amplify (or redirect to apex)
- `sage.medcore.tech` → Amplify app **sage-realtor**
- `melia.medcore.tech` → Amplify app **melia-contractor** (repo folder: `forge-contractor-app`)

## 2. Amplify — landing (`nodemon-site`)

1. Connect repo / branch (`nodemon` or `main`).
2. Root directory: `nodemon-site` (or repo root if this folder is the repo).
3. Build spec: `amplify.yml` (static — serves `medcore/` as site root).
4. Attach custom domain: `medcore.tech`, `www.medcore.tech`.

No environment variables required for the landing page.

## 3. Amplify — Sage (`sage-realtor-app`)

1. Connect repo / branch.
2. Root directory: `sage-realtor-app`.
3. Build spec: `amplify.yml` (`npm ci` + `npm run build`).
4. Attach custom domain: `sage.medcore.tech`.
5. **Environment variables** (Amplify console → Environment variables):

   | Key | Value |
   |-----|-------|
   | `VITE_API_BASE_URL` | `https://api.hearthmeet.com` |
   | `VITE_ELEVENLABS_DEMO_AGENT_ID` | *(your demo agent id, if used)* |

`public/_redirects` handles SPA routing (`/* → /index.html`).

## 3b. Amplify — Melia (`forge-contractor-app`)

Same steps as Sage:

1. Root directory: `forge-contractor-app`
2. Custom domain: `melia.medcore.tech`
3. Env: `VITE_API_BASE_URL=https://api.hearthmeet.com`

## 4. Backend env (EC2 / production `.env`) — required for browser + emails

API code stays the same; add origins and Sage app URL on the **server**:

```bash
# Append medcore origins (keep existing hearthmeet origins)
ALLOWED_ORIGINS=https://medcore.tech,https://www.medcore.tech,https://sage.medcore.tech,https://melia.medcore.tech

# Stripe checkout return URLs, password reset links, appointment emails
AIREALTOR_APP_URL=https://sage.medcore.tech
MELIA_APP_URL=https://melia.medcore.tech
ELEVENLABS_MELIA_AGENT_ID=   # optional; falls back to ELEVENLABS_AIREALTOR_AGENT_ID
```

Restart the backend after updating `.env`.

## 5. Local development

```bash
# Landing (port 8080)
cd nodemon-site && npx serve medcore -p 8080

# Sage (port 5174)
cd sage-realtor-app && cp .env.example .env.local && npm install && npm run dev
```

# Melia (port 5175)
cd forge-contractor-app && cp .env.example .env.local && npm install && npm run dev
```

Open http://localhost:8080 (landing), http://localhost:5174 (Sage), http://localhost:5175 (Melia).

## Alternative: Sage at `/sage` on the same host

If you prefer `https://medcore.tech/sage` instead of a subdomain, Sage needs `base: '/sage/'` in Vite, `BrowserRouter basename="/sage"`, and a combined deploy that copies `sage-realtor-app/dist/*` into `medcore/sage/`. Subdomain is simpler and is the default in this setup.
