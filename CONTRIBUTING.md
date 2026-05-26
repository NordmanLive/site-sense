# Contributing to site-sense

Thank you for your interest in contributing! Here's how to get started.

## Development Setup

```bash
git clone https://github.com/NordmanLive/site-sense.git
cd site-sense
npm install
npm run build
npm test        # 10 tests, <1s
```

## Making Changes

1. Fork the repo and create a branch from `main`
2. Make your changes
3. Run `npm run build && npm test` — all tests must pass
4. Submit a pull request

## What to Contribute

- **Bug fixes** — especially for specific browser/portal edge cases
- **Portal testing** — capture reports from Azure Portal, GitHub, ADO, etc.

## Code Style

- TypeScript strict mode
- No unnecessary comments (code should be self-explanatory)

## Security

If you find a security vulnerability, **do not open a public issue**. See [SECURITY.md](SECURITY.md) for responsible disclosure instructions.

## License

By contributing, you agree that your contributions will be licensed under the Apache-2.0 License.

## Releasing a New Version

1. Update `CHANGELOG.md` with the new version's entry under a new `## [x.y.z]` heading.
2. Update the `version` field in `package.json` and `extension/manifest.json` to match.
3. Push a tag: `git tag vX.Y.Z && git push origin vX.Y.Z`
4. The [publish workflow](.github/workflows/publish.yml) automatically builds, zips, and publishes to the Chrome Web Store.
5. Create a GitHub Release from the tag — paste the new CHANGELOG section as the release notes.

### Chrome Web Store secrets (one-time setup)

The publish workflow requires four GitHub Actions secrets. You need a Chrome
Web Store developer account ($5 one-time registration fee) and a Google Cloud
project to obtain OAuth2 credentials.

| Secret | What it is |
|---|---|
| `CHROME_EXTENSION_ID` | The extension ID — `jhapajnoajjppmbgmfhfnoonkmgglklm` for site-sense |
| `CHROME_CLIENT_ID` | OAuth 2.0 client ID from Google Cloud Console |
| `CHROME_CLIENT_SECRET` | OAuth 2.0 client secret from Google Cloud Console |
| `CHROME_REFRESH_TOKEN` | A long-lived refresh token granted to your OAuth client |

#### Step 1 — Register the extension manually (first time only)

1. Sign in to the [Chrome Web Store Developer Dashboard](https://chrome.google.com/webstore/devconsole/).
2. Click **New item** → upload a zip produced by `npm run package:ext`.
3. Fill in the listing details (description, screenshots, category, privacy policy URL).
4. Submit for review. Once approved, the extension ID in the URL must match
   the `key` in `extension/manifest.json` (otherwise re-uploads will fail).

#### Step 2 — Create an OAuth client

1. Open the [Google Cloud Console](https://console.cloud.google.com/) and
   create (or pick) a project.
2. Enable the **Chrome Web Store API** under *APIs & Services → Library*.
3. Under *APIs & Services → Credentials*, create an **OAuth 2.0 client ID**
   of type **Desktop app**. Save the **Client ID** and **Client secret**.

#### Step 3 — Generate a refresh token

Run a one-time OAuth flow against your client to obtain a refresh token
scoped to `https://www.googleapis.com/auth/chromewebstore`. The
[`chrome-webstore-upload-keys`](https://github.com/fregante/chrome-webstore-upload-keys)
helper automates the flow:

```bash
npx chrome-webstore-upload-keys
```

It opens a browser window for consent, then prints the refresh token.

#### Step 4 — Set the GitHub secrets

```bash
gh secret set CHROME_EXTENSION_ID  --body 'jhapajnoajjppmbgmfhfnoonkmgglklm' -R NordmanLive/site-sense
gh secret set CHROME_CLIENT_ID     -R NordmanLive/site-sense   # paste when prompted
gh secret set CHROME_CLIENT_SECRET -R NordmanLive/site-sense   # paste when prompted
gh secret set CHROME_REFRESH_TOKEN -R NordmanLive/site-sense   # paste when prompted
```

Verify with `gh secret list -R NordmanLive/site-sense`. From this point on,
pushing a `vX.Y.Z` tag triggers the publish workflow automatically.

See the [Chrome Web Store API docs](https://developer.chrome.com/docs/webstore/using-api)
for the underlying flow.
