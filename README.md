# Contingent Labor Cost Modeler

A browser-based PERT cost estimation tool for contingent labor and SOW engagements.
Served as a static single-page app via Nginx in Docker. State persists in each
user's browser via localStorage — no database, no backend.

**Live:** https://est.galvoti.net

---

## What it does

- Models contingent labor costs using PERT methodology (Optimistic / Most Likely / Pessimistic)
- Per-role rate inputs — defaults to the blended PERT rate, supports locked rate card values per role
- Utilization % and duration (start/end month) per role
- Calculates cost across Monthly, Quarterly, Semi-Annual, and Annual time horizons with Opt/PERT/Pes columns
- Weighted blended rate across all roles shown in the summary bar
- Save, version, and load named estimates (browser localStorage)
- Shareable link — encodes the full estimate state into the URL
- Export quote as PDF, Excel (.xlsx), or CSV
- Download a standalone formula-driven Excel template workbook
- Print view via browser print dialog

---

## Deploy via Portainer

The app is deployed from GitHub via Portainer using a pre-built image on ghcr.io.

### Initial deploy

In Portainer on megaplex:

- **Stacks → Add stack → Repository**
- Repository URL: `https://github.com/traxeon/pert-calculator`
- Repository reference: `refs/heads/main`
- Compose path: `docker-compose.yml`
- Authentication: off (public repo)
- **Deploy the stack**

Traefik and dockdns pick up the labels automatically. The app is live at
https://est.galvoti.net once DNS propagates (TTL 300s).

### Updates

After pushing changes to GitHub:

1. Build and push the new image from your Mac:

```bash
docker build -t ghcr.io/traxeon/pert-calculator:latest .
docker push ghcr.io/traxeon/pert-calculator:latest
```

2. In Portainer: **Stacks → pert-calculator → Pull and redeploy**

---

## Image registry

The app image is published to GitHub Container Registry:

```
ghcr.io/traxeon/pert-calculator:latest
```

The image is public — no registry credentials needed in Portainer.

---

## Persistence model

State is stored in each user's browser via `localStorage` under two keys:

| Key | Contents |
|---|---|
| `pert_auto_v1` | Auto-saved current state (updates on every change) |
| `pert_saves_v1` | Array of named saved estimates |

Saved estimates support versioning — saving over an existing name prompts to
overwrite or save as a new version (v2, v3...).

Users can **Export JSON** to back up all saved estimates and **Import JSON** to
restore — useful when switching browsers or clearing storage.

---

## Security

- All user input is sanitized via `sanitizeState()` before touching the DOM
- All strings are HTML-escaped via `esc()` at point of render
- DOM construction uses `createElement` / `textContent` — no `innerHTML` with user data
- Content Security Policy meta tag restricts script sources
- `noopener,noreferrer` on all `window.open` calls
- PDF export uses Blob URL instead of `document.write`

---

## Code scanning

CodeQL analysis runs on every push and weekly via GitHub Actions.
Results: https://github.com/traxeon/pert-calculator/security/code-scanning

To suppress a known false positive at the line level:
```javascript
const win = window.open(_pdfUrl, '_blank'); // lgtm[js/open-redirect]
```

---

## File structure

```
pert-calculator/
├── .github/
│   ├── workflows/
│   │   └── codeql.yml         # CodeQL scanning workflow
│   └── codeql-config.yml      # CodeQL false positive suppressions
├── app/
│   └── index.html             # Entire app — self-contained SPA
├── nginx/
│   └── nginx.conf             # Nginx server block config
├── Dockerfile
├── docker-compose.yml
└── README.md
```
