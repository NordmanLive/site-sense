# Changelog

All notable changes to site-sense are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [0.1.0] — Initial public release

First public release of site-sense — a Chrome/Edge extension + MCP server
that gives AI coding CLIs read-only access to your active browser tab.

### Added

#### MCP server (`bridge/src/index.ts`)
- **`site_sense_capture`** tool — returns the active tab's accessibility tree
  and a screenshot. Supports two modes:
  - **`compact`** (default) — interactive elements + landmarks + JPEG (quality
    50). Typical payload ~90 KB. Tool description biases models to always
    start here.
  - **`full`** — entire DOM + PNG. ~15× larger (~1.4 MB). Reserved for cases
    where compact is insufficient.
- **`site_sense_status`** tool — reports native-host connection state and
  session approval.

#### Extension (Manifest V3, TypeScript)
- Three-layer pipeline:
  - **Inject** (page MAIN world) — walks the DOM and builds the accessibility tree.
  - **Content** (isolated world) — origin-validated postMessage relay.
  - **Background** (service worker) — native messaging, session state, screenshots.
- Two permission modes:
  - **Default** — `activeTab` per page; user clicks the extension icon (or
    opens the popup) to grant access.
  - **All-sites** — opt-in via popup toggle; registers content scripts
    globally via `chrome.scripting.registerContentScripts`. Revoked when the
    CLI session ends.
- Popup auto-injects the content script into the active tab when reopened in
  an already-approved session, so newly opened windows don't need a fresh
  "Allow" click.
- Deterministic extension ID `jhapajnoajjppmbgmfhfnoonkmgglklm` (derived from
  the public key in `manifest.json`).

#### Native messaging bridge
- Thin native host registered via `npm run setup -- --browser chrome|edge`.
- MCP server ↔ native host communication via Unix domain socket
  (`${os.tmpdir()}/site-sense/bridge.sock`). No network traffic.
- Per-PID sockets for multi-session support.

#### Tooling and CI
- `npm run build` — TypeScript + 4-bundle Vite build (background, content,
  inject, popup) + static asset copy.
- `npm test` — vitest, 10 tests under `bridge/`, <1 s.
- `npm run zip:ext` / `npm run package:ext` — produces `site-sense.zip` for
  Chrome Web Store upload.
- GitHub Actions CI on push/PR to `main`: typecheck + build + tests on Node 22.
- GitHub Actions publish workflow on `v*.*.*` tag push: zips the extension
  and uploads to the Chrome Web Store via
  `fregante/chrome-webstore-upload-action@v2`.

#### Platform support
- **macOS** and **Linux**.
- **Chrome** and **Edge**.

### Security

- No network connections from extension or bridge — all transport is
  stdio/Unix-socket and Chrome native messaging.
- No persistent storage — capture state lives in memory and is cleared when
  the CLI disconnects.
- No write operations — site-sense never clicks, types, or modifies pages.
- Form values are skipped and URL tokens stripped from captures.
- See [`SECURITY.md`](SECURITY.md) for the full threat model.

### Known limitations

- Windows installer not implemented — `npm run setup` throws on `win32`.
- Firefox not supported.
- Extension-side code currently has no automated tests; coverage is limited
  to the MCP bridge.

[0.1.0]: https://github.com/NordmanLive/site-sense/releases/tag/v0.1.0
