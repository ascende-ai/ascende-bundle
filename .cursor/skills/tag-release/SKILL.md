---
name: tag-release
description: Runs the Ascende desktop bundle Git tag release on ascende-bundle—pins SHAs in desktop-release.json, verifies pre-release checklist (including ascende-frontend package.json version aligned with the tag for electron-builder artifact names), creates and pushes a v* semver tag to trigger GitHub Actions (macOS/Windows installers + GitHub Release). Use when cutting a desktop release, tagging ascende-bundle, shipping Electron installers, electron-updater metadata, or when the user invokes /tag-release or tag-release.
---

# Tag release (Ascende desktop bundle)

## Scope

This workflow applies to **`ascende-ai/ascende-bundle`**: pushing a tag matching **`v*`** runs [`.github/workflows/release-desktop.yml`](.github/workflows/release-desktop.yml), which clones pinned commits of **ascende-frontend** (`ascende-ai/ascende-fronted`), **ascende-deepagent**, and **jurisprudence-search**, builds PyInstaller + Next standalone + Electron, then attaches artifacts to a GitHub Release.

## App version vs bundle Git tag (installers)

The **Git tag** on **ascende-bundle** (e.g. `v1.0.13`) only names the release and selects pins. It does **not** flow into **electron-builder**.

**Artifact filenames** (`.dmg`, `.zip`, `.exe`, blockmaps) and much of the packaged app metadata come from **`ascende-frontend` `package.json`**: the **`version`** field (and **`productName`**), e.g. `ascende.ai-1.0.13-mac-arm64.dmg`.

**Before cutting a desktop release:**

1. Set **`"version"`** in **`ascende-frontend/package.json`** to the same semver as the bundle tag, **without** the `v` (tag `v1.0.13` → `"1.0.13"`).
2. Run **`npm install --package-lock-only`** (or equivalent) so **`package-lock.json`** stays in sync if CI uses `npm ci`.
3. Commit and push **ascende-fronted**, then set the **`ascende-frontend`** `ref` in **`desktop-release.json`** to that commit **before** tagging the bundle.

If **`package.json`** lags the tag, the GitHub Release can show **`v1.0.13`** while assets still contain **`1.0.11`** in their names.

## Before you change anything

1. Confirm the **target tag** (semver, e.g. `v1.2.3`). Tags must start with **`v`**.
2. Identify the **commits** to ship from each source repo (full **40-char SHAs** preferred).

## Step-by-step

**1. Bump app version (ascende-frontend) if needed**

Align **`package.json` `version`** with the semver you will tag on **ascende-bundle**, commit to **ascende-fronted**, then continue to pins.

**2. Update pins**

Edit **`desktop-release.json`** at the bundle repo root. For each of `ascende-frontend`, `ascende-deepagent`, and `jurisprudence-search`, set `ref` to the commit SHA (or branch only if you intentionally accept drift).

**3. Pre-release checklist** (from bundle README)

- [ ] **`ascende-frontend` `package.json` `version`** matches the intended bundle tag semver (see **App version vs bundle Git tag** above); **`package-lock.json`** updated if needed.
- [ ] **`desktop-release.json`** refs point at the builds you intend to ship (including the **ascende-fronted** commit that contains that **`version`** bump).
- [ ] **`BUNDLE_CHECKOUT_TOKEN`** is set on **ascende-bundle** (fine-grained PAT, **Contents: Read** on the three private source repos). Without it, CI fails at repo verification.
- [ ] **`package-lock.json`** exists in **ascende-frontend** and **jurisprudence-search** (CI uses `npm ci`).
- [ ] If you recently changed Python imports or Next standalone layout: PyInstaller still produces `ascende-backend` / `.exe`; jurisprudence still has `.next/standalone/server.js` after copy steps (see bundle README for `outputFileTracingRoot` notes).

**4. Commit pins (if needed)**

Commit and push **`desktop-release.json`** (and any other intentional changes) to the branch you release from (typically **`main`**), so the tag points at a commit that contains the correct pins.

**5. Create and push the tag**

From the **ascende-bundle** clone, on the correct commit:

```bash
git fetch origin
git checkout main   # or your release branch
git pull origin main
git tag vX.Y.Z
git push origin vX.Y.Z
```

Use an annotated tag only if your team standard requires it; the workflow keys off **`refs/tags/v*`** either way.

**6. After push**

- Watch **Actions** on **ascende-bundle** for the **Release desktop bundle** workflow.
- Confirm the **GitHub Release** for `vX.Y.Z` contains `.dmg`, `.zip`, `.exe`, `.blockmap`, and `latest*.yml` (electron-updater).

## Agent behavior

- Prefer **full SHAs** in `desktop-release.json` unless the user explicitly wants a branch pin.
- Before tagging, confirm **`ascende-frontend` `package.json` `version`** matches the numeric semver of the bundle tag; if the user only bumped the tag, flag the **installer filename mismatch** and offer to bump **`package.json`**, refresh the lockfile, push **ascende-fronted**, and repin **`desktop-release.json`**.
- If the user names a version without `v`, normalize to **`v` + semver** for the git tag.
- Do **not** push tags or secrets without explicit user confirmation; prepare diffs, checklist status, and exact commands.
- The frontend GitHub slug in JSON is **`ascende-ai/ascende-fronted`** (historical name); local folder may be `ascende-frontend`.

## Local parity (optional)

To reproduce CI locally: mirror the bundle README “Local parity” section—place the backend binary under frontend `python-backend/`, copy jurisprudence standalone into `jurisprudence-standalone/`, then `npm run build` in the frontend repo.
