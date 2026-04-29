# AGENTS.md

Quick reference for the DevLog site. Read `src/public/devlog.json` for data.

## Commands

```bash
cd src
npm install
npm run dev      # Dev server at localhost:5173
npm run build   # Outputs to src/dist/
npm run preview # Preview production build
```

## Tech Stack

- Vue 3 + Composition API + `<script setup>`
- Vuetify 4 (auto-import via vite-plugin-vuetify)
- Vite 8 (base path: `/devlog/`)

## Dev Log Data

Data lives in `src/public/devlog.json`:

```json
{
  "id": 1,
  "date": "2026-03-14",
  "title": "Entry Title",
  "content": "Description",
  "tags": ["apps", "launch"],
  "link": "https://example.com",
  "images": ["/devlog/assets-updates/image-name.png"],
  "commits": 15
}
```

### Images

1. Download from source (GitHub, Play Store, etc.)
2. Save to `src/public/assets-updates/` (create if needed)
3. Reference as `/devlog/assets-updates/filename.ext`
4. Commit images to main branch first, then update devlog

### Tags

| Tag      | Color | Description                      |
|----------|-------|----------------------------------|
| apps     | Blue  | App releases or updates         |
| launch   | Green | New launches                    |
| code     | Amber | Code-related                    |
| update   | Orange| General updates                 |
| content  | Purple| Content creation                |
| 3d-print | Cyan  | 3D printing                     |
| design   | Pink  | Design work                     |

## Deploy

All deploys stage to `/tmp/` first. Uses `git checkout public` branch for deployment.

### Devlog-only

```powershell
# On main: update src/public/devlog.json
Copy-Item src/public/devlog.json /tmp/
git checkout public
Copy-Item /tmp/devlog.json .
git add devlog.json
git commit -m "Update devlog"
git push
git checkout main
```

### Assets

```powershell
# On main: add to src/public/assets-updates/
Copy-Item -Recurse src/public/assets-updates /tmp/
git checkout public
Copy-Item -Recurse /tmp/assets-updates .
git add assets-updates
git commit -m "Update assets"
git push
git checkout main
```

### Full (code changes)

```powershell
# On main
cd src; npm install; npm run build
Copy-Item -Recurse src/dist /tmp/
git checkout public
Copy-Item -Recurse /tmp/dist/* .
git add -A
git commit -m "Update site: $(Get-Date -Format 'yyyy-MM-dd')"
git push
git checkout main
```