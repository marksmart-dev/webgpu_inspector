# CI/CD Workflows

This directory contains GitHub Actions workflows for building, packaging, and deploying the WebGPU Inspector browser extension.

---

## Workflows

### `ci.yml`
**Trigger:** Push to `main`/`master`, pull requests, and tags `v*`

**What it does:**
1. Installs Node.js 20 and project dependencies (`npm install`)
2. Builds the extension for Chrome and Firefox (`npm run build`)
3. Validates that manifest versions match the `VERSION` file
4. Verifies critical build output files exist
5. Packages each extension into a `.zip` (excludes source maps)
6. Uploads artifacts for download
7. Runs `web-ext lint` on the Firefox package for validation

**Artifacts produced:**
- `webgpu-inspector-chrome` — Chrome extension zip
- `webgpu-inspector-firefox` — Firefox extension zip
- `extensions-build` — Full build output directories

### `release.yml`
**Trigger:** Push of a tag starting with `v` (e.g., `v1.0.2`)

**What it does:**
- Runs the full build and packaging pipeline
- Creates a GitHub Release automatically
- Attaches both Chrome and Firefox extension zips to the release
- Generates release notes from commit history

**Usage:**
```bash
git tag -a v1.0.2 -m "Release v1.0.2"
git push origin v1.0.2
```

### `deploy-chrome.yml`
**Trigger:** Manual (`workflow_dispatch`) with a tag input

**What it does:**
- Builds and packages the Chrome extension from the specified tag
- Publishes to the Chrome Web Store using the `chrome-extension-upload` action

**Requirements:**
The following secrets must be configured in **GitHub Repository Settings → Secrets and variables → Actions**:
- `CHROME_CLIENT_ID` — OAuth client ID from Google Cloud Console
- `CHROME_CLIENT_SECRET` — OAuth client secret
- `CHROME_REFRESH_TOKEN` — OAuth refresh token
- `CHROME_EXTENSION_ID` — Extension ID from Chrome Web Store

See: https://developer.chrome.com/docs/webstore/using-api

### `deploy-firefox.yml`
**Trigger:** Manual (`workflow_dispatch`) with a tag input

**What it does:**
- Builds and packages the Firefox extension from the specified tag
- Signs and submits to Firefox Add-ons (AMO) using `web-ext`

**Requirements:**
The following secrets must be configured in **GitHub Repository Settings → Secrets and variables → Actions**:
- `FIREFOX_API_KEY` — JWT issuer from AMO Developer Hub
- `FIREFOX_API_SECRET` — JWT secret from AMO Developer Hub

See: https://extensionworkshop.com/documentation/develop/web-ext-command-reference/#web-ext-sign

---

## Recommended Release Flow

1. **Bump version** in `VERSION` file (and update `CHANGELOG.md`)
2. **Commit and push:**
   ```bash
   git add VERSION CHANGELOG.md
   git commit -m "Bump version to 1.0.3"
   git push origin main
   ```
3. **Tag the release:**
   ```bash
   git tag -a v1.0.3 -m "Release v1.0.3"
   git push origin v1.0.3
   ```
4. **Wait for CI:** The `ci.yml` and `release.yml` workflows will run automatically. The GitHub Release will be created with both extension packages attached.
5. **Deploy to stores:** Go to **Actions → Deploy to Chrome Web Store** (or Firefox) → **Run workflow**, enter `v1.0.3`, and dispatch.

---

## Notes

- Source maps (`*.map`) are excluded from store packages but kept in build artifacts for debugging.
- Safari extension builds are not automated (requires macOS + Xcode).
- `package-lock.json` is currently gitignored. For fully reproducible CI builds, consider committing it to the repo.
- Firefox AMO may require source code submission because the extension uses minified code. The public GitHub repo URL can be provided during submission.
