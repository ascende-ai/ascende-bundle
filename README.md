# Ascende desktop release orchestrator

Remote: [github.com/ascende-ai/ascende-bundle](https://github.com/ascende-ai/ascende-bundle).

Single place to **pin versions** of `ascende-frontend`, `ascende-deepagent`, and `jurisprudence-search`, then build macOS and Windows installers on every **version tag** (`v*`).

The **jurisprudence-search** Next.js app is produced as a **standalone bundle only to ship inside the desktop app**. It is not deployed as a public website; end users consume it through the Electron shell only.

## Before you tag

1. Edit `desktop-release.json`: pin each `ref` to a **full commit SHA** (recommended) or branch name.
2. Ensure all three source repositories are readable by `GITHUB_TOKEN` (public repos work with the default token; private repos need a PAT with `contents: read` on each repo and pass it to `actions/checkout` `token:`).
3. **Auto-update:** desktop builds and `electron-updater` expect GitHub Releases on **`ascende-ai/ascende-bundle`** (configured in `ascende-frontend`).

## Trigger

```bash
git tag v1.0.0
git push origin v1.0.0
```

The workflow builds PyInstaller `ascende-backend`, Next standalone for jurisprudence, then Electron; uploads job artifacts and attaches them to a GitHub Release for the tag.

## Local parity (optional)

Roughly mirrors CI: build the backend binary into `ascende-frontend/python-backend/`, copy the jurisprudence standalone tree into `ascende-frontend/jurisprudence-standalone/`, then from the frontend run `npm run build`.
