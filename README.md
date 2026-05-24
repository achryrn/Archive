# HomeNode

I built HomeNode as my personal self-hosted web server, the backend that powers everything I run on my own hardware. Music streaming, synced lyrics, authentication, file serving, a portfolio, and study tools, all without depending on any third-party platform.

> **Private deployment.** This repository documents the architecture and setup process. The live instance is not open for public registration.

---

## What it does

- **Music streaming**, Serves my personal music library with transcoding, queue management, and offline playlist support
- **Lyrics sync**, Real-time synced lyrics tied to playback position, backed by a local database with a pipeline for adding new tracks
- **Authentication**, JWT-based session system with role-based access control and rate limiting
- **LapLink**, Persistent agent protocol for pairing my laptop to the server and issuing remote commands
- **Portfolio**, Serves my resume and gallery with per-user access control
- **Study tools**, Backend for a personal spaced-repetition and study session tracker
- **Admin analytics**, Request logs and a user activity dashboard

---

## Tech stack

| Layer | Technology |
|---|---|
| Runtime | Python 3.12 |
| Framework | Flask + Flask-SocketIO |
| Auth | JWT (custom implementation) |
| Database | SQLite (per-feature databases) |
| Media | FFmpeg (transcoding), Jellyfin (optional) |
| File sharing | Samba (optional) |
| Reverse proxy | Nginx |
| Process manager | systemd |

---

## Project structure

```
homenode/
├── server.py
├── auth_system.py
├── music_database.py
├── art_lookup.py
├── lyrics_pipeline.py
├── lyrics_db_extension.py
├── admin_analytics.py
├── routes/
│   ├── flask_auth_routes.py
│   ├── music_routes.py
│   ├── lyrics_routes.py
│   ├── laplink_routes.py
│   ├── portfolio_routes.py
│   ├── study_routes.py
│   └── backend_offline_playlist_route.py
└── templates/
    ├── index.html
    ├── music.html
    ├── study.html
    ├── laplink.html
    ├── resume.html
    ├── mobile.html
    └── admin.html
```

---

**Required variables:**

| Variable | Description |
|---|---|
| `SECRET_KEY` | Flask session signing key |
| `PARTNER_TOKEN` | API token for the partner client |
| `LAPLINK_AGENT_TOKEN` | Token for the LapLink agent handshake |
| `LAPLINK_PASSPHRASE_HASH` | Hashed passphrase for LapLink pairing |
| `ALLOWED_ORIGINS` | Comma-separated list of allowed CORS origins |
| `BASE_DIRECTORY` | Absolute path to the file storage root |
| `MUSIC_LIBRARY_PATH` | Absolute path to the music library folder |
| `ART_STORE_PATH` | Absolute path to the album art cache folder |

**Optional variables:**

| Variable | Description |
|---|---|
| `REGISTRATION_TOKEN` | If set, required in the register request body. Leave unset to disable registration. |
| `PORT` | Listening port |
| `FLASK_DEBUG` | Must be `false` in production |
| `GALLERY_BASE` | Absolute path to the gallery folder |
| `JELLYFIN_URL` | Jellyfin base URL |
| `JELLYFIN_ENABLED` | Set to `false` to disable Jellyfin integration |
| `SAMBA_HOST` | Samba host for file browsing |
| `SAMBA_ENABLED` | Set to `false` to disable Samba integration |

Generate secrets with:
```bash
python3 -c "import secrets; print(secrets.token_urlsafe(48))"
```

---

## Running locally

**Requirements:** Python 3.12+, FFmpeg, SQLite3

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
python3 server.py
```

---

## Production deployment

I run this with systemd and Nginx. systemd manages the process lifecycle and restarts on failure. Nginx sits in front as a reverse proxy and handles TLS termination. `ALLOWED_ORIGINS` should be set to your domain only.

## Related

- [Lyricryn](https://github.com/achryrn/lyricryn), The Android app that connects to this backend
