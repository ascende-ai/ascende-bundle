# ascende-bundle

Desktop **release orchestrator** for Ascende: one place to **pin versions** (commits) of **ascende-frontend**, **ascende-deepagent**, and **jurisprudence-search**, and produce **macOS** and **Windows** installers on every `v*` **tag**.

Upstream repository: [github.com/ascende-ai/ascende-bundle](https://github.com/ascende-ai/ascende-bundle).

The **jurisprudence-search** app is built as **Next.js standalone** only to be **embedded in Electron** (`extraResources`), not as a separate public website.

## Before you tag

1. Edit **`desktop-release.json`**: set each `ref` to a **full commit SHA** (recommended) or a branch name for reproducible builds.
2. **Private repositories:** add the **`BUNDLE_CHECKOUT_TOKEN`** secret to this GitHub repository.
   - **Settings → Secrets and variables → Actions → New repository secret** → name `BUNDLE_CHECKOUT_TOKEN`.
   - **Fine-grained PAT** (recommended): [github.com/settings/tokens?type=beta](https://github.com/settings/tokens?type=beta) → resource owner **ascende-ai** → only the repos you need → **Repository contents: Read-only** → paste the token into the secret.
   - The default Actions `GITHUB_TOKEN` **cannot** read other private repos; without this secret, the workflow fails at the verification step.
3. **Auto-update:** desktop builds and **electron-updater** expect **GitHub Releases** on this repo (**ascende-bundle**), as configured in **ascende-frontend**.
4. **Visibility:** **ascende-bundle** is usually **public** so the release feed is reachable without a per-user token. Source repos can stay private via `BUNDLE_CHECKOUT_TOKEN`.

> In `desktop-release.json`, the frontend `repository` field points at the GitHub slug **`ascende-ai/ascende-fronted`** (historical repo name); your local checkout folder may still be named `ascende-frontend`.

## Trigger a release

```bash
git tag v1.0.0
git push origin v1.0.0
```

The workflow builds the **PyInstaller** binary (`ascende-backend`), the jurisprudence **standalone** bundle, then **Electron**; uploads artifacts and attaches them to the GitHub Release for the tag.

## Pre-release checklist

1. **Pinned SHAs** in `desktop-release.json` for anything shipped to users.
2. **PyInstaller:** if the app launches but crashes on import, adjust `scripts/ascende-backend.spec` in **ascende-deepagent** (`hiddenimports` / `collect_all`). Local repro: `pip install . pyinstaller` then `pyinstaller scripts/ascende-backend.spec`.
3. **Next standalone:** the copied tree must include `server.js` at the root; if it does not, fix **`outputFileTracingRoot`** in **jurisprudence-search**.
4. **Lockfiles:** `package-lock.json` must exist in **ascende-frontend** and **jurisprudence-search** (CI uses `npm ci`).
5. **electron-updater:** metadata (`latest-mac.yml`, etc.) produced under `ascende-frontend/release/` should be uploaded with the installers.

## Local parity (optional)

Mirror CI: place the backend binary under `ascende-frontend/python-backend/`, copy the jurisprudence standalone tree into `ascende-frontend/jurisprudence-standalone/`, then from the frontend repo run `npm run build`.

## Ecosystem

| Repository | Role in the bundle |
|------------|-------------------|
| **ascende-frontend** | Electron app, UI, updater |
| **ascende-deepagent** | API + PyInstaller spec |
| **jurisprudence-search** | Embedded web UI (standalone) |
