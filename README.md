# Immich Drop – Home Assistant Add-on

Collect photos and videos from anyone into your Immich server using simple invite links. This add-on packages the upstream Immich Drop web uploader so you can run it directly on Home Assistant OS/Supervisor.

- Upstream project: https://github.com/Nasogaa/immich-drop
- This add-on’s manifest: [config.yaml](immich-drop/config.yaml:1)
- Service launcher: [run](immich-drop/rootfs/etc/s6-overlay/s6-rc.d/immich-drop/run:1)
- Container build: [Dockerfile](immich-drop/Dockerfile:1)
- Changelog: [CHANGELOG](immich-drop/CHANGELOG.md:1)

Branding
- The add-on includes icons used by Home Assistant Supervisor:
  - [icon.png](immich-drop/icon.png:1)
  - [logo.png](immich-drop/logo.png:1)
  These are placed where Supervisor expects them (the add-on folder), so they appear automatically in the Add-on Store UI.

## What it does

Immich Drop lets you share an upload link (or QR code) with guests so they can send photos/videos straight to your Immich server. Admins can log in to create and manage invite links. You can optionally add uploads to a default album, protect invites with a password, and enable a public landing page if you like.

Highlights
- Invite links: public-by-URL, one‑time or multi‑use
- Manage links: enable/disable, edit expiry, copy, QR, delete
- Optional album add (auto-creates if missing)
- Duplicate prevention (local SHA‑1 cache + Immich bulk-check)
- Progress queue with retry
- Mobile-friendly, dark mode
- Optional chunked uploads for very large files

## Requirements

- Home Assistant OS (Supervisor) recommended
- An Immich server and API key
- Network access from HA to Immich (often same LAN/VPN)
- Recommended: HTTPS on your public reverse proxy, if exposed

## Install

1) Add this repository to your Add-on Store:
- Settings → Add-ons → Add-on Store → menu (⋮) → Repositories
- Add your GitHub repo URL for this add-on (the repository root that contains [repository.json](repository.json:1) and the [immich-drop](immich-drop/config.yaml:1) folder)

2) Open “Immich Drop”, click Install.

3) Configure options (see below), click Save.

4) Start the add-on, then click Open Web UI.

Tip: After saving changes, restart the add-on for them to take effect.

## Quick config (recommended starting point)

Example:
```yaml
immich_base_url: "https://immich.example.com/api"
immich_api_key: "PASTE-YOUR-TOKEN"       # asset.upload; album.* scopes for albums
immich_album_name: ""                    # leave blank to not auto-add to an album
public_upload_page_enabled: false        # invite links still work without this
public_base_url: ""                      # optional, used in invite link you copy
chunked_uploads_enabled: false           # enable for very large files
chunk_size_mb: 95
log_level: info
session_secret: ""                       # optional; auto-generated if blank (recommended to set a strong random value in production)
```

Open the Web UI:
- If public_upload_page_enabled = true, the landing page is at “/”.
- Admin pages: Login → Menu → Create invite → Copy link / Show QR.

## How invites work (simple flow)

- Admin visits the Web UI, logs in, creates an invite (optionally one‑time, expiry, password, album).
- Share the invite URL or the QR code with guests.
- Guests open the invite, pick files, and upload with progress feedback.
- If you configured an album, successful uploads get added automatically.

## Duplicate behaviour

The add-on keeps a local dedupe cache to avoid re-uploading the exact same file:
- If a file was uploaded before by this add-on instance, it is flagged “duplicate” even if you deleted the asset in Immich later (this prevents accidental re-queues).
- To completely reset local dedupe history, stop the add-on and remove its local DB at /data/state.db (Supervisor persists /data for add-ons). A UI button to clear this cache can be added later.

## Large file support (chunked)

- Enable chunked_uploads_enabled and set chunk_size_mb to help bypass typical 100 MB limits on some proxies.
- The client switches to chunked mode only for files larger than the configured size.

## Updates and releases

- This add-on tracks upstream Immich Drop releases automatically with CI.
- When a new upstream release is detected, the add-on version is bumped to that exact tag and the add-on is pinned to it. You’ll see an Update button in the Add-on Store shortly after.
- We also publish a GitHub Release in this repository that includes the upstream release notes for easy reference.

## Troubleshooting

- I can open the page, but uploads fail quickly:
  - Confirm immich_base_url includes “/api”, e.g. https://your-immich.example.com/api
  - Verify your Immich API key scopes (asset.upload is required; album.* for albums)
  - Check that HA can reach Immich’s address/port on the network
- Upload reports “duplicate” immediately:
  - That’s the local cache doing its job. If you want to allow re-uploads, clear /data/state.db (the add-on’s local state) and restart the add-on.
- GPS/location missing on some photos:
  - If the logs show GPS EXIF present before upload, the bytes were forwarded as-is. Missing GPS after upload often originates from the device/picker path (e.g., certain Android camera/share flows can sanitize EXIF) or from downstream processing. Try selecting from gallery/files rather than “Take Photo” or verify device share settings.

## Privacy and security

- Invite links are public-by-URL. Share only with intended recipients.
- The default landing page at “/” is disabled unless you enable public_upload_page_enabled.
- The API key is server-side only; the browser never sees it.
- For Internet exposure, run behind HTTPS and set public_base_url.

## Credits

- Immich Drop upstream: https://github.com/Nasogaa/immich-drop
- Home Assistant Community add-on patterns inspired by projects such as the Example add-on and community add-ons (e.g. alexbelgium).

## Developer notes (optional)

- Configuration and options schema: [config.yaml](immich-drop/config.yaml:21)
- Service start script (exports options via bashio and starts uvicorn): [run](immich-drop/rootfs/etc/s6-overlay/s6-rc.d/immich-drop/run:1)
- Add-on Docker build (base image, deps, pin to upstream ref): [Dockerfile](immich-drop/Dockerfile:1)
- CI for upstream release sync, tagging, and release publishing: [.github/workflows/sync-upstream.yml](.github/workflows/sync-upstream.yml:1)
