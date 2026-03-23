# AGENTS.md

This document provides instructions for LLM agents working with the David R. Comba Lareu DevLog site.

## Repository Structure

```
davidc.github.io/
├── src/                    # Source code (Vite + Vue + Vuetify)
│   ├── public/
│   │   └── devlog.json    # Dev log entries data
│   └── src/
│       └── App.vue        # Main application component
└── .git/
```

## Dev Log JSON Structure

The dev log entries are stored in `src/public/devlog.json`. Each entry has the following structure:

```json
{
  "id": 1,
  "date": "2026-03-14",
  "title": "Entry Title",
  "content": "Description of the update",
  "tags": ["update", "launch"],
  "link": "https://example.com",
  "images": ["https://example.com/image1.jpg"],
  "commits": 15
}
```

### Field Descriptions

| Field      | Type     | Required | Description                                        |
|------------|----------|----------|----------------------------------------------------|
| `id`       | number   | Yes      | Unique identifier for the entry                    |
| `date`     | string   | Yes      | Date in `YYYY-MM-DD` format                        |
| `title`    | string   | Yes      | Title of the update                                |
| `content`  | string   | Yes      | Detailed description                               |
| `tags`     | string[] | Yes      | Array of tags (see available tags below)            |
| `link`     | string   | No       | External link (icon shown if provided)             |
| `images`   | string[] | No       | Array of image URLs (displayed as thumbnails)      |
| `commits`  | number   | No       | Number of commits associated with this update      |

### Available Tags

| Tag         | Color   | Description                          |
|-------------|---------|--------------------------------------|
| `apps`      | Blue    | Application releases or updates      |
| `launch`    | Green   | New launches (counts as "ships")     |
| `code`      | Amber   | Code-related updates                 |
| `update`    | Orange  | General updates                      |
| `content`   | Purple  | Content creation                     |
| `3d-print`  | Cyan    | 3D printing projects                 |
| `design`    | Pink    | Design work                          |

### Adding a New Entry

1. Generate a unique `id` (use the highest existing id + 1)
2. Use the current date in `YYYY-MM-DD` format
3. Add appropriate tags from the available list
4. Set `commits` to reflect the number of commits for this update
5. Include `link` and `images` if applicable

Example of adding a new entry:

```json
[
  // ... existing entries ...
  {
    "id": 4,
    "date": "2026-03-15",
    "title": "New Feature Implemented",
    "content": "Added a new feature that improves user experience.",
    "tags": ["code", "apps"],
    "link": "https://github.com/shadowofsoul/repo",
    "images": ["https://example.com/screenshot.png"],
    "commits": 8
  }
]
```

## Building and Deploying

### Prerequisites

```bash
cd src
npm install
```

### Building the Site

```bash
cd src
npm run build
```

This creates the built files in `src/dist/`.

### Deploying to Public Branch

1. Build the project:
   ```bash
   cd src && npm run build
   ```

2. Copy built files to root and update public branch:
   ```bash
   git checkout public
   cp -r src/dist/* .
   git add -A
   git commit -m "Update site with latest changes"
   ```

3. Switch back to main:
   ```bash
   git checkout main
   ```

### Full Deploy Script

```bash
#!/bin/bash
cd src
npm run build
git checkout public
cp -r src/dist/* .
git add -A
git commit -m "Update site: $(date +%Y-%m-%d)"
git checkout main
```

## GitHub API Integration

If a GitHub repo is provided, you can use the `gh` CLI to fetch commit data.

### Get Commit Activity

```bash
gh api repos/{owner}/{repo}/stats/commit_activity
```

Example:
```bash
gh api repos/shadowofsoul/myproject/stats/commit_activity
```

This returns weekly commit counts:
```json
[
  {
    "week": 1678320000,
    "total": 15,
    "days": [0, 3, 5, 2, 3, 2, 0]
  }
]
```

### Get Recent Commits

```bash
gh api repos/{owner}/{repo}/commits --paginate --jq '.[].sha'
```

### Get Commits Since a Date

```bash
gh api "repos/{owner}/{repo}/commits?since=2026-03-01" --jq 'length'
```

### Get Contributor Stats

```bash
gh api repos/{owner}/{repo}/stats/contributors
```

### Get Repository Languages

```bash
gh api repos/{owner}/{repo}/languages
```

### Useful gh queries for devlog data

```bash
# Count commits in last month
gh api "repos/{owner}/{repo}/commits?since=$(date -d '1 month ago' +%Y-%m-%d)" --jq 'length'

# Get latest release tag
gh api repos/{owner}/{repo}/releases/latest --jq '.tag_name'

# List all releases
gh api repos/{owner}/{repo}/releases --jq '.[].name'
```

## Activity Grid Behavior

The activity grid shows:
- Each colored block represents a day with at least one update
- Block color matches the primary tag of that day's update
- Clicking a block filters to show that day's entry first, then other entries from the same month
- Activity grid automatically shows the most recent year with data

## Filter Behavior

- **Tags filter**: Multi-select, shows entries matching any selected tag
- **Activity grid**: Click to filter by specific date
- **Stats**: Update dynamically based on filtered results
  - Ships = count of entries with "launch" tag
  - Commits = sum of all commit counts
