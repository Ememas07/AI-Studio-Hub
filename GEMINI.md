# AI-Studio-Hub Agent Instructions

## Architecture Overview
- **Hub Repository:** `Ememas07/AI-Studio-Hub` (Public) — hosted on a raspberry pi with dynamic dns at `https://marc-mas.ddns.net/`.
- **Sub-Repositories:** Public AI Studio projects (`Ememas07/<repo-name>`) cloned over HTTPS (`--depth 1`), compiled, and bundled into `_site/<repo-name>/` by `AI-Studio-Hub`'s GitHub Actions workflow (`deploy.yml`).
- **Upward Chaining:** Each sub-repo has `.github/workflows/notify-hub.yml` and a `HUB_DISPATCH_TOKEN` secret (Fine-grained PAT with `Contents: Read and write` scoped strictly to `Ememas07/AI-Studio-Hub`). Pushing to `main` in any sub-repo fires a `repository_dispatch` (`subrepo-updated`) event to rebuild and redeploy `AI-Studio-Hub`.

---

## Command: "add repo <url>"

When the user asks to add a new repository (e.g., `add repo https://github.com/Ememas07/<repo-name>`), execute these steps automatically using `gh` CLI and `git`:

### 1. Parse & Inspect the New Sub-Repository
- Extract `<repo-name>` from the provided URL (`Ememas07/<repo-name>`).
- Ensure the repository is **Public** so `AI-Studio-Hub` can clone it without a read token (`gh repo view Ememas07/<repo-name> --json visibility`).
- Inspect the repo via `gh api` or a temporary shallow clone to determine:
  - **Title & Description:** Read `README.md` or `package.json` for a clean display title and 1-sentence summary.
  - **Framework & Base Path:** Confirm it matches a builder supported by `AI-Studio-Hub`'s `.github/workflows/deploy.yml` (Vite/React/Three.js via `package.json`, Angular via `angular.json`, or plain static HTML/JS). If `vite.config.*` has a hardcoded `base: '/'`, change it to `base: './'` (or remove it so `--base=./` takes effect) and push the fix.

### 2. Configure Upward Chaining in the Sub-Repository
- Disable any standalone GitHub Pages site on the sub-repo (`gh api -X DELETE repos/Ememas07/<repo-name>/pages 2>/dev/null || true`) and remove any redundant standalone `.github/workflows/deploy.yml` if present.
- Set `HUB_DISPATCH_TOKEN` on `Ememas07/<repo-name>`:
  - Read `HUB_DISPATCH_TOKEN` from the local `.env` file in `AI-Studio-Hub` if available; otherwise ask the user to paste the token once, then run:
    ```bash
    gh secret set HUB_DISPATCH_TOKEN -R Ememas07/<repo-name> --body "<token>"
    ```
- Add `.github/workflows/notify-hub.yml` to `main` in `Ememas07/<repo-name>`:
  ```yaml
  name: Notify AI-Studio-Hub

  on:
    push:
      branches: ["main"]

  jobs:
    trigger-hub:
      runs-on: ubuntu-latest
      steps:
        - name: Trigger AI-Studio-Hub Rebuild
          run: |
            curl -X POST \
              -H "Accept: application/vnd.github+json" \
              -H "Authorization: Bearer ${{ secrets.HUB_DISPATCH_TOKEN }}" \
              https://api.github.com/repos/Ememas07/AI-Studio-Hub/dispatches \
              -d '{"event_type":"subrepo-updated","client_payload":{"repo":"${{ github.repository }}"}}'