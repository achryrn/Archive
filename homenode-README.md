# HomeNode

A self-hosted personal web server built with Flask. Serves as the backend for music streaming, lyrics sync, authentication, file serving, portfolio, and study tools - all running on your own hardware.

> **Private deployment.** This repository documents the architecture and setup process. The live instance is not open for public registration.

---

## What it does

- **Music streaming** - Serves a personal music library with transcoding, queue management, and offline playlist support
- **Lyrics sync** - Real-time synced lyrics tied to playback position, backed by a local database with a pipeline for new tracks
- **Authentication** - JWT-based session system with role-based access (admin, partner, user), bot shielding, and rate limiting
- **LapLink** - Persistent agent protocol for laptop-to-server pairing and remote control
- **Portfolio** - Serves resume and gallery with per-user access whitelisting
- **Study tools** - Backend for a personal spaced-repetition and study session tracker
- **Admin analytics** - Request logs, user activity dashboard, and security stream

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
| Reverse proxy | Nginx (recommended) |
| Process manager | systemd |

---

## Project structure

```
homenode/
├── server.py                          # App factory, startup validation, SocketIO init
├── auth_system.py                     # DatabaseManager, AuthenticationManager, UserManager
├── music_database.py                  # MusicDatabaseManager - library indexing & metadata
├── art_lookup.py                      # Album art resolution (local + remote fallback)
├── lyrics_pipeline.py                 # Lyrics fetch, parse, and DB insertion pipeline
├── lyrics_db_extension.py             # Lyrics DB schema and query helpers
├── admin_analytics.py                 # AnalyticsManager - request logging & aggregation
├── bot_shield.py                      # Request fingerprinting and bot detection
├── security_stream.py                 # Real-time security event emitter (SocketIO)
├── routes/
│   ├── flask_auth_routes.py           # /api/auth/* and /admin/* endpoints
│   ├── music_routes.py                # /api/music/* - streaming, queue, search
│   ├── lyrics_routes.py               # /api/lyrics/* - synced & static lyrics
│   ├── laplink_routes.py              # /api/laplink/* - agent pairing protocol
│   ├── portfolio_routes.py            # /api/portfolio/* - resume, gallery
│   ├── study_routes.py                # /api/study/* - sessions, cards, stats
│   └── backend_offline_playlist_route.py  # Offline playlist manifest endpoint
└── templates/
    ├── index.html                     # Main SPA shell
    ├── music.html                     # Music player view
    ├── study.html                     # Study session view
    ├── laplink.html                   # LapLink agent dashboard
    ├── resume.html                    # Portfolio/resume view
    ├── mobile.html                    # Mobile-optimized view
    └── admin.html                     # Admin analytics dashboard
```

---

## Configuration

All configuration is done through environment variables. Copy `.env.example` to `.env` and fill in every value before starting.

```bash
cp .env.example .env
```

**Required variables:**

| Variable | Description |
|---|---|
| `SECRET_KEY` | Flask session signing key. Minimum 32 characters. |
| `PARTNER_TOKEN` | API token for the partner client (LapLink). Minimum 16 characters. |
| `LAPLINK_AGENT_TOKEN` | Separate token for the LapLink agent handshake. |
| `LAPLINK_PASSPHRASE_HASH` | SHA-256 hash of the LapLink pairing passphrase. |
| `ALLOWED_ORIGINS` | Comma-separated list of allowed CORS origins. |
| `BASE_DIRECTORY` | Absolute path to the file storage root. |
| `MUSIC_LIBRARY_PATH` | Absolute path to the music library folder. |
| `ART_STORE_PATH` | Absolute path to the album art cache folder. |

**Optional variables:**

| Variable | Default | Description |
|---|---|---|
| `REGISTRATION_TOKEN` | *(empty - registration disabled)* | If set, required in the register request body. |
| `PORT` | `5000` | Listening port. |
| `FLASK_DEBUG` | `false` | Never set to `true` in production. |
| `GALLERY_BASE` | - | Absolute path to the gallery/backup folder. |
| `GALLERY_WHITELIST` | - | Comma-separated usernames allowed gallery access. |
| `PARTNER_ALLOWED_USERNAMES` | - | Comma-separated usernames with partner-level access. |
| `JELLYFIN_URL` | `http://localhost:8096` | Jellyfin base URL. |
| `JELLYFIN_ENABLED` | `true` | Set to `false` to disable Jellyfin integration. |
| `SAMBA_HOST` | `localhost` | Samba host for file browsing. |
| `SAMBA_ENABLED` | `true` | Set to `false` to disable Samba integration. |
| `MOVIE_SITE_URL` | `http://localhost:8096` | URL proxied for the movie site. |

Generate secrets with:
```bash
python3 -c "import secrets; print(secrets.token_urlsafe(48))"
```

Generate the LapLink passphrase hash with:
```bash
python3 -c "import hashlib; print(hashlib.sha256(b'your-passphrase').hexdigest())"
```

---

## Running locally

**Requirements:** Python 3.12+, FFmpeg, SQLite3

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
# Fill in .env values
python3 server.py
```

---

## Production deployment

The recommended setup is systemd + Nginx.

**systemd unit** (`/etc/systemd/system/homenode.service`):
```ini
[Unit]
Description=HomeNode Web Server
After=network.target

[Service]
User=youruser
WorkingDirectory=/path/to/homenode
EnvironmentFile=/path/to/homenode/.env
ExecStart=/path/to/homenode/.venv/bin/python3 server.py
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now homenode
```

**Nginx** should proxy to `127.0.0.1:5000` and handle TLS termination. Set `ALLOWED_ORIGINS` to your domain only.

---

## Security notes

- The `.env` file is in `.gitignore` and must never be committed.
- `FLASK_DEBUG` must be `false` in production - debug mode exposes a code execution console.
- `REGISTRATION_TOKEN` should be set or registration should be left disabled unless you intend to allow new users.
- Bot shielding and rate limiting are active on all auth endpoints.
- All secrets in `.env.example` are placeholders - rotate them before any deployment.

---

## Related

- [Lyricryn](https://github.com/yourusername/lyricryn) - The Android/Flutter client that connects to this backend.
