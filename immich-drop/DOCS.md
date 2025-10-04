# Immich Drop – Home Assistant Add-on

Web uploader for Immich with invite links, optional public upload page, duplicate prevention, progress, and album add. This add-on packages the upstream Immich Drop Python app to run under Home Assistant OS/Supervisor using the Community Add-ons base image and s6-overlay.

- Service launcher (s6): see [run](immich-drop/rootfs/etc/s6-overlay/s6-rc.d/immich-drop/run:1)
- App config loader: [python.load_settings()](immich-drop-master/app/config.py:33)

---

## Installation

1) Add this repository to Home Assistant Add-on Store
- Settings → Add-ons → Add-on store
- Menu (top-right) → Repositories
- Add repository URL: https://github.com/Chaoscontrol/addon-immich-drop

2) Install
- Open the “Immich Drop” add-on from the store and click Install.

3) Configure options (see below) and Start the add-on.

4) Open Web UI
- Web UI will be available at http://<home-assistant-host>:<port>/ (default port 8080)
- Or click “Open Web UI” from the add-on page.

---

## Configuration

Example

```yaml
immich_base_url: "https://immich.example.com/api"
immich_api_key: "PASTE-YOUR-TOKEN"
immich_album_name: ""                 # optional; default blank means no album
public_upload_page_enabled: false     # whether / serves the public uploader; /invite/* always public-by-URL
public_base_url: "https://drop.example.com"  # optional, for absolute invite URLs
chunked_uploads_enabled: false
chunk_size_mb: 95
log_level: info
session_secret: ""                    # optional; leave blank to auto-generate
```

Option details
- immich_base_url (required): Must include /api, e.g., https://immich.example.com/api
- immich_api_key (required): Immich API key. Minimum scopes:
  - asset.upload
  - album.create, album.read, albumAsset.create (if using albums)
- immich_album_name (optional): If set, uploads are also added to this album (auto-creates if not existing)
- public_upload_page_enabled (optional, default false): When true, the root path / shows the public uploader. Invite links remain public-by-URL regardless.
- public_base_url (optional): Used to build absolute URLs for invite links when returned by the API (fallback: request base URL)
- chunked_uploads_enabled (optional): Enable client-side chunked uploads to bypass 100MB proxy limits (e.g., Cloudflare)
- chunk_size_mb (optional, default 95): Per-chunk size used by the client when chunked uploads are enabled
- log_level: trace|debug|info|notice|warning|error|fatal (mapped to the app’s LOG_LEVEL)
- session_secret (optional): If omitted, a secure random value is generated at startup
  - Recommendation (production): set a strong random value to keep session cookies unguessable (matches upstream release notes “Production tip: set SESSION_SECRET”).

Network
- Port 8080/tcp is exposed by default. You can change the host port mapping on the add-on info page if needed.

Persistence
- The add-on persists its local dedupe DB at /data/state.db (managed by Supervisor automatically). Chunk assembly uses /data/chunks.

---

## Upgrade notes (v0.5.0-beta alignment)

- This add-on follows upstream v0.5.0-beta. Upstream marks this release “Stable · Feature & maintenance”.
- No schema changes or database changes are required by upstream; your add-on options remain the same.
- Breaking changes noted upstream are about Manage Links markup/classes (frontend). The add-on is unaffected, but if you use custom browser user-styles, they might need an update.
- Security: upstream recommends setting `SESSION_SECRET`. This add-on already supports it via options. Strongly recommended for production.

## Usage

- Admin flow:
  1. Open the Web UI → Login with Immich credentials → Menu
  2. Create invite links (optionally one‑time / expiry / album)
  3. Share the invite link or QR code with recipients

- Guest flow:
  1. Open the invite link (no login)
  2. Drop or select files → View progress and result per item

- Optional default uploader:
  - If you set public_upload_page_enabled=true, the landing page / allows uploads (often combined with immich_album_name).

---

## How it works

- The add-on launches Uvicorn to serve the FastAPI backend and static frontend. The s6 longrun service exports add-on options as environment variables and starts the server: see [run](immich-drop/rootfs/etc/s6-overlay/s6-rc.d/immich-drop/run:1).
- The application reads configuration exclusively from environment variables using [python.load_settings()](immich-drop-master/app/config.py:33).
- Duplicate prevention: local SQLite cache (state.db) + optional Immich bulk-upload-check.
- Progress updates are streamed to the browser via WebSocket per session.
- Optional album integration automatically creates/fetches an album and adds new uploads to it.
- Optional chunked uploads split large files into parts to bypass common reverse proxy limits.

---

## Testing checklist

After installing and configuring:
1) Start the add-on and open Logs
2) Verify configuration echo (base URL masked token), then “Starting uvicorn…” appears
3) Open Web UI
4) Ping banner should show connectivity OK (if Immich API key and URL valid)
5) Login → Menu → Create Invite
6) Open invite URL in a separate private browser window and upload a test image/video
7) Confirm the item appears in Immich; if an album is configured, confirm it’s added

---

## Troubleshooting

- Blank page or connection issues:
  - Ensure immich_base_url includes /api
  - Verify Immich is reachable from HA network and TLS/CA is valid
  - If using a proxy/tunnel (e.g., Cloudflare), consider enabling chunked uploads

- 401/403 on albums:
  - The Immich API key may be missing album.* scopes; update the key

- Duplicate detected immediately:
  - The local cache records checksums and device IDs; try a different file or clear /data/state.db (will reset local dedupe history)

- Port already in use:
  - Change the host port binding in the add-on page (the container internal port remains 8080)

Logs
- Add-on logs provide detailed info; set log_level to debug for extra detail

---

## Notes

- This add-on does not watch a host directory; it is a web uploader that streams files directly to Immich via its API. The /media path in Home Assistant is not used by this add-on.
- Ingress is not enabled by default; use direct Web UI access on the configured port. Ingress support can be added in a later release.

---

## Maintainer

- Add-on repository: https://github.com/Chaoscontrol/addon-immich-drop
- Upstream project: https://github.com/Nasogaa/immich-drop