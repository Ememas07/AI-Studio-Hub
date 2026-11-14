# AI-Studio-Hub Agent Instructions

## Architecture Overview
- **Hub Repository:** `Ememas07/AI-Studio-Hub` (Public) — self-hosted on a Raspberry Pi behind Caddy (`https://marc-mas.ddns.net/`).
- **Sub-Repositories:** Public AI Studio projects (`Ememas07/<repo-name>`) listed in `repos.txt`.
- **Deployment Pipeline:** The Raspberry Pi runs `/usr/local/bin/sync-site.sh` via cron every 2 minutes. It automatically pulls `AI-Studio-Hub`, clones/updates every repository in `repos.txt`, runs Vite/Angular builds if needed, symlinks outputs into `/var/www/public/<repo-name>`, injects the global favicon, and updates Caddy's lowercase URL redirects. No GitHub Actions workflows or dispatch tokens are used.

---

## Command: "add repo <url>"

When the user asks to add a new repository (e.g., `add repo https://github.com/Ememas07/<repo-name>`), execute these steps automatically using `gh` CLI and `git`:

### 1. Parse & Inspect the New Sub-Repository
- Extract `<repo-name>` from the provided URL (`Ememas07/<repo-name>`).
- Verify the repository is **Public** so the Raspberry Pi can clone it without credentials (`gh repo view Ememas07/<repo-name> --json visibility`).
- Inspect the repo via `gh api` or a temporary shallow clone to determine:
  - **Title & Description:** Read `README.md` or `package.json` for a clean display title and 1-sentence summary.
  - **Framework & Base Path:** Confirm it uses a supported structure (Vite/React/Three.js via `package.json`, Angular via `angular.json`, or plain static HTML/JS).
  - **Relative Asset Paths:** If `vite.config.*` has a hardcoded `base: '/'`, change it to `base: './'` (or remove it so `--base=./` takes effect) and push the fix so assets resolve under `/<repo-name>/`.
  - **Back to Hub Button:** Ensure the sub-repository's UI includes a clearly visible button or link to return to the main hub page (`https://marc-mas.ddns.net/` or `/`), adding one cleanly to the layout/HUD if missing and pushing the change.

### 2. Point the Sub-Repository's GitHub Website URL to the Server
- Set the GitHub repository's homepage/website field to `https://marc-mas.ddns.net/<repo-name>/`:
  ```bash
  gh repo edit Ememas07/<repo-name> --homepage "[https://marc-mas.ddns.net/](https://marc-mas.ddns.net/)<repo-name>/"