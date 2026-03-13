# ♪ Music App

A modern Android music player that streams audio from Google Drive. Built with clean MVVM architecture, dark UI, and smooth animations.

---

## 📱 Screenshots

> _Add your screenshots here_

| Splash | Song List | Player | Add Song |
|--------|-----------|--------|----------|
| ![Splash]() | ![Songs]() | ![Player]() | ![Add]() |

---

## ✨ Features

- 🎵 **Stream audio from Google Drive** — download-then-play for instant seek and offline support
- 💾 **Smart caching** — songs download once and play from local storage forever after
- 🎨 **Dark UI** — deep purple theme with glowing rings, gradients and animated equalizer
- 🎬 **Animated splash screen** — YouTube-style scale + pulse + equalizer bar animation
- ▶️ **Full player controls** — play, pause, next, previous, seekbar with time labels
- 🔔 **Mini player** — persistent bar with progress, prev/play/next controls
- ➕ **Add songs** — post new songs via Google Drive file ID
- 🔄 **Offline-first** — Room cache always shown first, refreshed from network in background
- 🔍 **Chucker** — built-in HTTP inspector (debug builds only)

---

## 🏗️ Architecture

```
Single Activity (MainActivity)
│
├── NavHostFragment
│   ├── SongListFragment    ← browse & tap to play
│   ├── PlayerFragment      ← full-screen player
│   └── AddSongFragment     ← add new song via Drive ID
│
├── MiniPlayer bar          ← persistent, always visible
└── BottomNavigationView
```

### Stack

| Layer | Technology |
|-------|-----------|
| Language | Kotlin |
| Architecture | MVVM + Repository |
| DI | Hilt |
| Navigation | Jetpack Navigation Component (single activity) |
| Network | Retrofit 2 + OkHttp |
| Local DB | Room |
| Player | ExoPlayer (Media3) |
| Image loading | Coil |
| Async | Kotlin Coroutines + StateFlow |
| HTTP Inspector | Chucker (debug only) |

---

## 📂 Project Structure

```
app/src/main/java/com/example/musicapp/
├── MusicApp.kt                  ← @HiltAndroidApp
├── api/
│   └── AudioApiService.kt       ← Retrofit endpoints
├── db/
│   ├── MusicDatabase.kt         ← Room database
│   └── SongDao.kt               ← DAO queries
├── di/
│   └── AppModule.kt             ← Hilt modules (OkHttp, Retrofit, Room)
├── model/
│   └── Models.kt                ← Song, PlayerState, UiState, DownloadState
├── repository/
│   └── MusicRepository.kt       ← offline-first data logic
├── viewmodel/
│   └── MusicViewModel.kt        ← StateFlow hub for all UI state
└── ui/
    ├── MainActivity.kt          ← ExoPlayer host, mini player observer
    ├── SplashActivity.kt        ← animated splash
    ├── SongAdapter.kt           ← RecyclerView adapter with Coil + badges
    └── fragments/
        ├── SongListFragment.kt
        ├── PlayerFragment.kt
        └── AddSongFragment.kt
```

---

## 🚀 Getting Started

### Prerequisites

- Android Studio Hedgehog or newer
- Android SDK 34
- Minimum device: Android 7.0 (API 24)
- A running instance of the backend API

### Clone & Run

```bash
git clone https://github.com/YOUR_USERNAME/music-app.git
cd music-app
```

Open in Android Studio → **Run ▶**

### Configure API URL

In `app/src/main/java/com/example/musicapp/di/AppModule.kt`, update the base URL:

```kotlin
.baseUrl("http://YOUR_SERVER_IP:3000/")
```

---

## 🌐 API Reference

Base URL: `http://YOUR_SERVER_IP:3000/`

### GET `/products/audio-list`

Returns all saved songs.

```json
{
  "success": true,
  "data": [
    {
      "_id": "69b3c7e2c5839a12f7b6d8a6",
      "title": "My Song",
      "fileId": "1x_rQhrVK3qomuqyXT8xQVFgXANV5DLT_",
      "url": "https://drive.google.com/uc?export=download&id=..."
    }
  ]
}
```

### POST `/products/add-audio`

Add a new song.

**Request body:**
```json
{
  "title": "My Song",
  "fileId": "1x_rQhrVK3qomuqyXT8xQVFgXANV5DLT_"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Audio saved successfully",
  "data": { "_id": "...", "title": "...", "fileId": "...", "url": "..." }
}
```

---

## ☁️ Adding Songs via Google Drive

1. Upload your audio file to Google Drive
2. Right-click → **Share** → set to **"Anyone with the link"** → Done
3. Right-click → **Get link** → Copy
4. Extract the **File ID** from the link:
   ```
   https://drive.google.com/file/d/FILE_ID_IS_HERE/view?usp=sharing
   ```
5. Open the app → tap **+** → enter title + file ID → **Add Song**

> ⚠️ The file **must** be set to "Anyone with the link". Private files return an HTML login page instead of audio.

---

## 🎵 How Playback Works

```
Tap song
    │
    ├── File cached locally? ──► YES ──► play instantly from local file
    │
    └── NO
        │
        ▼
    Show ↓ spinner on song row
        │
        ▼
    Download from Drive URL + confirm=t (bypasses consent page)
        │
        ▼
    Validate: file > 10KB? (small = HTML error page, not audio)
        │
        ▼
    Save to /data/.../files/{songId}.mp3
        │
        ▼
    Show ✓ Cached badge
        │
        ▼
    ExoPlayer plays local file (no network, no buffering, instant seek)
```

---

## 🔍 Debugging with Chucker

On debug builds, every API call is intercepted by **Chucker**.

- Pull down the notification drawer while the app is running
- Tap the Chucker notification to see all HTTP requests/responses
- Useful for verifying: song list response, add song payload, Drive download URL

---

## 🎨 UI Components

| File | Purpose |
|------|---------|
| `activity_splash.xml` | Animated splash — rings + music note + equalizer bars |
| `fragment_song_list.xml` | Dark song list with song count + cached count |
| `fragment_player.xml` | Full-screen player with art, seekbar, time labels |
| `layout_mini_player.xml` | Persistent mini player with progress bar |
| `item_song.xml` | Song row card with Coil art + cached/downloading badge |
| `fragment_add_song.xml` | Add song form with outlined dark fields |

---

## 🛠️ Tech Decisions

**Why download-then-play instead of streaming?**
Google Drive doesn't support HTTP range requests needed for audio streaming. Downloading first gives instant seek, offline playback, and avoids ExoPlayer `UnrecognizedInputFormatException` caused by Drive's HTML consent redirect pages.

**Why single activity?**
ExoPlayer lives in `MainActivity` and survives all fragment transactions. If it lived in a fragment it would be recreated on every navigation event, interrupting playback.

**Why SharedFlow for seekTo?**
If `seekTo` updated `PlayerState.progress`, every state emission would re-seek ExoPlayer even when the user didn't drag the seekbar. `SharedFlow` fires exactly once per user drag.

**Why offline-first Repository?**
Room cache is shown immediately on every launch. Network refresh happens in background. Even with no internet, the app shows previously loaded songs.

---

## 📦 Dependencies

```gradle
// Architecture
androidx.navigation:navigation-fragment-ktx:2.7.7
androidx.lifecycle:lifecycle-viewmodel-ktx:2.7.0
androidx.room:room-runtime:2.6.1

// Network
com.squareup.retrofit2:retrofit:2.9.0
com.squareup.okhttp3:okhttp:4.12.0

// DI
com.google.dagger:hilt-android:2.50

// Player
androidx.media3:media3-exoplayer:1.2.1

// UI
io.coil-kt:coil:2.5.0
com.google.android.material:material:1.11.0

// Debug
com.github.chuckerteam.chucker:library:4.0.0 (debug only)
```

---

## 📄 License

```
MIT License — free to use, modify and distribute.
```

---

<p align="center">Built with ❤️ using Kotlin + Jetpack</p>
