# Immich Drop – Home Assistant Add-on

A Home Assistant OS add-on that packages the Immich Drop Uploader (FastAPI + static UI) to let guests upload photos/videos to your Immich server via invite links. Duplicate prevention, optional album association, optional public uploader landing page, and chunked uploads to bypass 100MB reverse proxy limits.

- Upstream app entry: [app.app:app](immich-drop-master/app/app.py:40)
- Config loader: [load_settings()](immich-drop-master/app/config.py:33)
- Add-on service launcher: [run](immich-drop/rootfs/etc/s6-overlay/s6-rc.d/immich-drop/run:1)
- Add-on image build: [Dockerfile](immich-drop/Dockerfile:1)
- Add-on manifest: [config.yaml](immich-drop/config.yaml:1)
- Add-on docs: [DOCS.md](immich-drop/DOCS.md:1)

Note: This add-on is a web uploader; it does not watch a Home Assistant folder. Files are uploaded by clients directly to Immich via its API.

## Repository structure

- repository.json
- immich-drop/
  - build.yaml
  - config.yaml
  - Dockerfile
  - DOCS.md
  - rootfs/
    - etc/s6-overlay/s6-rc.d/immich-drop/{run,type}
    - etc/s6-overlay/s6-rc.d/user/contents.d/immich-drop
- immich-drop-master/ (vendored upstream source for local reference and dev)

The Dockerfile actually clones the latest upstream at build time to /opt/immich_drop by default, ensuring you get the maintained app. You can pin to a specific revision later.

## Quick start (Home Assistant OS/Supervisor)

1) Push this repository to GitHub (see below).
2) In Home Assistant, go to Settings → Add-ons → Add-on store → Menu → Repositories, and add your repository URL.
3) Install the “Immich Drop” add-on, configure options, Start, and Open Web UI.

See [DOCS.md](immich-drop/DOCS.md:1) for full configuration and troubleshooting.

## Options overview

```yaml
immich_base_url: "https://immich.example.com/api"
immich_api_key: "PASTE-YOUR-TOKEN"
immich_album_name: ""
public_upload_page_enabled: false
public_base_url: "https://drop.example.com"
chunked_uploads_enabled: false
chunk_size_mb: 95
log_level: info
session_secret: ""
```

The s6 service exports these options as environment variables before launching uvicorn. The app reads them via [python.load_settings()](immich-drop-master/app/config.py:33).

## Local development notes

- No special dev requirements for the add-on itself. The upstream project includes a dev runner [uvicorn.run()](immich-drop-master/main.py:16) for reload, but the add-on uses uvicorn without reload under s6.
- The dedupe DB is persisted at /data/state.db inside the add-on container (Supervisor handles persistence).

## Contributing

- Adjust schema or defaults in [config.yaml](immich-drop/config.yaml:1).
- Update service behavior in [run](immich-drop/rootfs/etc/s6-overlay/s6-rc.d/immich-drop/run:1).
- Improve docs in [DOCS.md](immich-drop/DOCS.md:1).

## License

- Upstream immich-drop is MIT (see its repo for details).
- This add-on scaffold is MIT unless otherwise noted.