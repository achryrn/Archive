# Lyricryn

An Android music player built with Flutter. Connects to a self-hosted [HomeNode](https://github.com/yourusername/homenode) backend to stream your personal music library with synced lyrics, offline downloads, and a persistent notification player.

> **Private backend.** This app is designed for use with a personal HomeNode instance. There is no public server.

---

## Features

- Stream music from your HomeNode library over LAN or internet
- Real-time synced lyrics that scroll with playback
- Download tracks for offline playback
- Background playback with media notification controls (play, pause, next, prev)
- Library browsing - songs, playlists, albums
- Search across the full library
- Track request queue - submit requests to the server
- Auth-gated - requires a HomeNode account
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
│   ├── main.dart                          # App entry point, provider setup
│   ├── theme/
│   │   └── app_theme.dart                 # Color scheme and typography
│   ├── models/
│   │   ├── song.dart                      # Song data model
│   │   └── playlist.dart                  # Playlist data model
│   ├── data/
│   │   └── mock_data.dart                 # Development stubs
│   ├── services/
│   │   ├── api_service.dart               # All HomeNode HTTP calls
│   │   ├── audio_handler.dart             # just_audio session wrapper
│   │   ├── audio_focus_manager.dart       # Android audio focus handling
│   │   ├── download_service.dart          # Track download and local storage
│   │   ├── notification_service.dart      # Playback notification setup
│   │   └── update_notification_service.dart  # In-app update prompts
│   ├── providers/
│   │   ├── auth_provider.dart             # Login state and token storage
│   │   ├── music_provider.dart            # Library data and queue
│   │   ├── player_provider.dart           # Playback state
│   │   ├── download_provider.dart         # Download queue state
│   │   ├── connectivity_provider.dart     # Online/offline detection
│   │   └── update_provider.dart          # App update check state
│   ├── screens/
│   │   ├── auth/
│   │   │   ├── auth_gate.dart             # Route guard
│   │   │   ├── login_screen.dart          # Login UI
│   │   │   └── register_screen.dart       # Registration UI
│   │   ├── home/
│   │   │   └── home_screen.dart           # Home feed
│   │   ├── library/
│   │   │   ├── library_screen.dart        # Full library browser
│   │   │   └── playlist_detail_screen.dart
│   │   ├── player/
│   │   │   └── player_screen.dart         # Full-screen now playing
│   │   ├── search/
│   │   │   └── search_screen.dart         # Search UI
│   │   ├── lyrics/
│   │   │   ├── lyrics_screen.dart         # Synced lyrics view
│   │   │   ├── lyrics_provider.dart       # Lyrics fetch and sync state
│   │   │   └── lyric_line.dart            # Individual lyric line widget
│   │   ├── request/
│   │   │   └── request_screen.dart        # Track request form
│   │   └── main_shell.dart               # Bottom nav shell
│   └── widgets/
│       ├── mini_player.dart               # Persistent bottom player bar
│       ├── song_tile.dart                 # Song list item
│       ├── artwork_widget.dart            # Album art with fallback
│       ├── playlist_card.dart             # Playlist grid card
│       └── add_to_playlist_sheet.dart     # Bottom sheet for playlist add
└── android/
    └── app/src/main/kotlin/com/lyricyrn/lyricyrn/
        ├── MainActivity.kt                # Flutter activity + method channels
        └── MusicNotificationService.kt    # Foreground service for media controls
```

---

## Configuration

The app connects to a HomeNode backend. The base URL is set in `lib/services/api_service.dart`.

For a development build pointing at a local server:
```dart
static const String baseUrl = 'http://192.168.x.x:5000';
```

For a release build, set this to your production domain with HTTPS.

There are no API keys or secrets stored in the app - authentication is handled via username/password login against the HomeNode auth endpoints, and the session token is stored in `shared_preferences`.

---

## Building

**Requirements:** Flutter SDK 3.x, Android SDK, a connected device or emulator.

```bash
flutter pub get
flutter run                    # debug on connected device
flutter build apk --release    # release APK
```

The release APK can be sideloaded or hosted on your HomeNode instance for OTA updates.

---

## Android permissions

Declared in `AndroidManifest.xml`:

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

- [HomeNode](https://github.com/yourusername/homenode) - The Flask backend this app connects to.
