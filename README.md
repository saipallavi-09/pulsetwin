# PulseTwin

## GitHub Pages deployment

The live URL `https://saipallavi-09.github.io/pulsetwin/` currently shows this README because GitHub Pages is publishing the repository root, which contains documentation and uploaded ZIP files rather than a built website.

This repository now includes `.github/workflows/deploy-pages.yml`. To deploy the frontend:

1. Upload the **extracted repository contents**, not `frontend.zip` or `backend.zip`. The repository root must contain `frontend/`, `backend/`, and `.github/`.
2. Push the contents to the `main` branch.
3. Open **Settings → Pages** in GitHub and set **Source** to **GitHub Actions**.
4. Push a new commit or run the **Deploy PulseTwin frontend to GitHub Pages** workflow from the Actions tab.
5. After the workflow completes, open `https://YOUR-USERNAME.github.io/YOUR-REPOSITORY/`.

The frontend uses hash routing, so routes such as the Ward View work on GitHub Pages without server-side rewrite rules. GitHub Pages hosts only the static frontend; the FastAPI backend must be deployed separately if live API data is required. Without a backend URL, the Ward View still renders its synthetic preview and interactive floor-plan demo.


Emergency-department digital twin: a React frontend and a FastAPI backend,
in one repo.

```
pulsetwin/
  frontend/     React + TypeScript + Vite (see frontend/README.md)
  backend/      FastAPI + queueing engine + simulation (see backend/README.md)
```

Each folder is independent — its own `package.json` / `requirements.txt` —
so each deploys as its own service, even though they live in one repo.

## Deploying

Because this is a monorepo, whichever hosts you use need to be told the
**root directory** is `backend/` or `frontend/`, not the repo root. Below are
the two most common free-tier options for each side. Use whichever hosts you
prefer — the key setting to look for on any host is "root directory" /
"base directory".

### Backend (FastAPI) → Render

1. On [render.com](https://render.com), New → Web Service → connect this repo.
2. **Root Directory:** `backend`
3. **Build Command:** `pip install -r requirements.txt`
4. **Start Command:** `uvicorn app.main:app --host 0.0.0.0 --port $PORT`
5. Deploy. Render gives you a URL like `https://pulsetwin-backend.onrender.com`.

### Frontend (Vite/React) → Vercel

1. On [vercel.com](https://vercel.com), New Project → import this repo.
2. **Root Directory:** `frontend`
3. Framework preset: Vite (auto-detected). Build command `npm run build`,
   output directory `dist` (both auto-filled).
4. **Environment Variable:** `VITE_API_BASE_URL` = your Render backend URL
   from above (e.g. `https://pulsetwin-backend.onrender.com`).
5. Deploy.

### After both are deployed

Open the Vercel URL — it should load the Command Center and pull live data
from the Render backend. If you see a "Couldn't reach the PulseTwin backend"
error, double-check `VITE_API_BASE_URL` has no trailing slash and that the
backend's Render logs show it started without errors.

## Local development

See `backend/README.md` and `frontend/README.md` for running each side on
your own machine before deploying.

## Upgraded demo flow

The current prototype implements the manager-facing decision loop described in the product brief:

1. **Command Center** shows live synthetic ward telemetry, pressure trend, acuity mix, capacity utilization, and active operational signals.
2. **Ask the twin** parses a natural-language question such as “What if patient arrivals increase by 30% tomorrow night?” into structured scenario parameters.
3. **Twin Optimizer** runs fast queueing screening, validates the shortlist with a discrete-event simulation, and ranks candidates with severity-aware waiting-time, utilization, throughput, and staffing-cost scoring.
4. **Recommendation** explains why the selected intervention is preferred, shows patient-impact deltas and guardrails, and can save the decision.
5. **Ward Twin** renders the simulated patient state by pipeline stage, event trace, virtual patient manifest, and applied resource utilization.
6. **Decision history** stores saved recommendations in the backend's prototype in-memory history store.

The prototype remains intentionally honest about its data boundary: it uses synthetic operational data and is a simulation-based digital twin, not a clinical system or a real-time hospital integration.

### Local demo

Run the backend from `backend/` with `uvicorn app.main:app --reload`, then run the frontend from `frontend/` with `npm run dev`. Set `VITE_API_BASE_URL` if the API is not served from the same origin.
