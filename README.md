# Service Area Sandbox (GitHub Pages + Cloudflare Worker)

Interactive map to add/remove up to 10 points and generate **combined isochrone bands** (30/45/60) at any time of day. Bands render as **non-overlapping rings** in a **single hue** so colors don't turn muddy, and your base map stays visible.

## Folder structure
```
service-area-sandbox/
├─ index.html          # Frontend (host on GitHub Pages)
└─ worker/
   └─ worker.js        # Cloudflare Worker proxy (keeps API keys safe)
```

## Why a proxy?
GitHub Pages is static. Calling TravelTime directly would expose your keys in the browser. The **Cloudflare Worker** stores your keys as **secrets** and forwards approved requests.

---

## Deploy the Cloudflare Worker (free)
1. Create a Cloudflare account → **Workers** → **Create Worker**.
2. Paste `worker/worker.js` into the editor.
3. In Worker **Settings → Variables** add two secrets:
   - `TRAVELTIME_APP_ID`
   - `TRAVELTIME_API_KEY`
4. Edit the **allowed origin** in `worker.js` to your GitHub Pages origin:
   - `https://YOUR-USERNAME.github.io`
5. **Deploy** and copy the Worker URL, e.g. `https://your-worker.ACCOUNT.workers.dev`.

> Optional local dev with Wrangler:
> ```bash
> npm i -g wrangler
> wrangler dev worker/worker.js
> # local URL will be printed; set it in index.html while testing
> ```

---

## Configure & run the frontend
1. Open `index.html` and set:
   ```js
   const WORKER_URL = "https://YOUR-WORKER.workers.dev";
   ```
2. To test locally:
   ```bash
   # in service-area-sandbox/
   python -m http.server 8000
   # open http://localhost:8000
   ```
3. For GitHub Pages:
   - Create a repo, push both `index.html` and `worker/worker.js` (worker file doesn't need to be hosted, but it's nice for reference).
   - Enable **Settings → Pages → Deploy from branch**, folder: **/ (root)**.
   - Visit `https://YOUR-USERNAME.github.io/REPO/`

---

## Using the sandbox
- Click on the map to add a point (max 10). Click a marker to remove it.
- Pick a time of day with the slider.
- Check which bands (30/45/60) you want.
- Press **Recalculate**.
- The app calls the Worker → TravelTime, unions polygons per band, and draws **non-overlapping rings** so colors don’t blend.

## Notes
- The Worker enforces CORS to the specified GitHub Pages origin.
- You can switch the **transportation type** in `worker.js` if needed.
- If your area is very large and calls get heavy, you can debounce the **Recalculate** button or add rate limits on the Worker.
