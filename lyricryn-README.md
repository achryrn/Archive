# Lyricryn

I built Lyricryn as the Android companion to my self-hosted [HomeNode](https://github.com/achryrn/homenode) backend. It streams my personal music library with synced lyrics, offline downloads, and a persistent notification player — no third-party streaming service involved.

> **Private backend.** This app connects to a personal HomeNode instance. There is no public server.

---

## Screenshots

| Home | Search | Request |
|---|---|---|
| ![Home screen showing recently played and recommended tracks](screenshots/screenshot_home.png) | ![Search screen with results for "Untuk"](screenshots/screenshot_search.png) | ![Request screen searching YouTube for a track to add to the server library](screenshots/screenshot_request.png) |

| Library | Downloads | Spotify Import |
|---|---|---|
| ![Library screen showing 847 songs](screenshots/screenshot_library.png) | ![Downloads tab showing 4 offline tracks](screenshots/screenshot_downloads.png) | ![Import from Spotify dialog with a playlist link pasted](screenshots/screenshot_playlist_import.png) |

---

## Features

- Stream from my HomeNode library over LAN or internet
- Real-time synced lyrics that scroll with playback position
- Download tracks for offline playback
- Background playback with media notification controls
- Library browsing across songs, playlists, albums
- Search across the full library
- Request screen — search YouTube and add tracks directly to the server library
- Import a Spotify playlist link to sync it to the server library automatically
- Auth-gated — requires a HomeNode account
- Auto-update notifications when a new APK is available on the server

---

## Tech stack

| | |
|---|---|
| Framework | Flutter 3.x |
| State management | Provider |
| Audio engine | just_audio |
| HTTP | Dio + http |
| Image caching | cached_network_image |
| Storage | shared_preferences, path_provider |
| Fonts | google_fonts |
| Notifications | Android foreground service (Kotlin) |

---

## Project structure

```
lyricryn/
├── lib/
│   ├── main.dart
│   ├── theme/
│   │   └── app_theme.dart
│   ├── models/
│   │   ├── song.dart
│   │   └── playlist.dart
│   ├── data/
│   │   └── mock_data.dart
│   ├── services/
│   │   ├── api_service.dart
│   │   ├── audio_handler.dart
│   │   ├── audio_focus_manager.dart
│   │   ├── download_service.dart
│   │   ├── notification_service.dart
│   │   └── update_notification_service.dart
│   ├── providers/
│   │   ├── auth_provider.dart
│   │   ├── music_provider.dart
│   │   ├── player_provider.dart
│   │   ├── download_provider.dart
│   │   ├── connectivity_provider.dart
│   │   └── update_provider.dart
│   ├── screens/
│   │   ├── auth/
│   │   │   ├── auth_gate.dart
│   │   │   ├── login_screen.dart
│   │   │   └── register_screen.dart
│   │   ├── home/
│   │   │   └── home_screen.dart
│   │   ├── library/
│   │   │   ├── library_screen.dart
│   │   │   └── playlist_detail_screen.dart
│   │   ├── player/
│   │   │   └── player_screen.dart
│   │   ├── search/
│   │   │   └── search_screen.dart
│   │   ├── lyrics/
│   │   │   ├── lyrics_screen.dart
│   │   │   ├── lyrics_provider.dart
│   │   │   └── lyric_line.dart
│   │   ├── request/
│   │   │   └── request_screen.dart
│   │   └── main_shell.dart
│   └── widgets/
│       ├── mini_player.dart
│       ├── song_tile.dart
│       ├── artwork_widget.dart
│       ├── playlist_card.dart
│       └── add_to_playlist_sheet.dart
└── android/
    └── app/src/main/kotlin/com/lyricyrn/lyricyrn/
        ├── MainActivity.kt
        └── MusicNotificationService.kt
```

---

## Configuration

The backend URL is set in `lib/services/api_service.dart`. For a development build use your local server address; for a release build use your production domain with HTTPS.

There are no API keys or secrets stored in the app. Authentication is handled via username/password login against the HomeNode auth endpoints, and the session token is stored in `shared_preferences`.

---

## Building

**Requirements:** Flutter SDK 3.x, Android SDK, a connected device or emulator.

```bash
flutter pub get
flutter run
flutter build apk --release
```

The release APK can be sideloaded or hosted on a HomeNode instance for OTA updates.

---

## Android permissions

| Permission | Reason |
|---|---|
| `INTERNET` | Stream music from the server |
| `FOREGROUND_SERVICE` | Background playback notification |
| `FOREGROUND_SERVICE_MEDIA_PLAYBACK` | Media notification category |
| `POST_NOTIFICATIONS` | Playback controls in notification shade |
| `READ_EXTERNAL_STORAGE` / `WRITE_EXTERNAL_STORAGE` | Offline download storage (Android < 10) |
| `RECEIVE_BOOT_COMPLETED` | Restore playback session on reboot |
| `WAKE_LOCK` | Keep CPU alive during background playback |

---

## Related

- [HomeNode](https://github.com/achryrn/homenode) — The Flask backend this app connects to
