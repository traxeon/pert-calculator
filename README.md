# PERT Staffing Estimator

ServiceNow contractor cost estimator using PERT methodology. Served as a static
single-page app via Nginx in Docker. State persists in each user's browser via
localStorage — no database, no backend.

Available at: https://est.galvoti.net

---

## Deploy via Portainer

### 1. Copy files to megaplex

```bash
scp -r pert-calculator/ ken@megaplex:/opt/stacks/pert-calculator
```

### 2. In Portainer

- Go to **Stacks → Add stack**
- Name it `pert-calculator`
- Select **Upload** and choose `docker-compose.yml`, or paste its contents
- Click **Deploy the stack**

Traefik and dockdns will pick up the labels automatically. The app will be live
at https://est.galvoti.net once DNS propagates (usually within the TTL — 300s).

---

## Updates

To redeploy after changing `index.html`:

```bash
# On megaplex — replace the file
cp index.html /opt/stacks/pert-calculator/app/index.html
```

Then in Portainer: **Stacks → pert-calculator → Editor → Update the stack**
(this triggers a rebuild since `build.context` points to the stack directory).

Or from the CLI on megaplex:

```bash
cd /opt/stacks/pert-calculator
docker compose up -d --build
```

---

## Persistence model

State is stored in each user's browser via `localStorage` under two keys:

| Key | Contents |
|---|---|
| `pert_auto_v1` | Auto-saved current state (updates on every change) |
| `pert_saves_v1` | Array of named saved estimates |

Users can also **Export JSON** to download a backup and **Import JSON** to
restore — useful if they switch browsers or clear storage.

---

## File structure

```
/opt/stacks/pert-calculator/
├── app/
│   └── index.html        # Entire app — self-contained SPA
├── nginx/
│   └── nginx.conf        # Nginx server block config
├── Dockerfile
├── docker-compose.yml
└── README.md
```
