# 🏗 Live Radio System — Architecture

## Overview

This system is a **Scheduling-First** radio broadcasting engine. It does **not** shuffle or serve tracks on demand. Instead, the Laravel backend strictly enforces a time-based schedule stored in the MySQL database, ensuring every listener hears the exact same audio at the exact same moment — a true live radio experience.

---

## System Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                        LINUX SERVER                         │
│                                                             │
│  ┌──────────────┐    file path    ┌────────────────────┐   │
│  │   Laravel    │ ──────────────► │      FFmpeg        │   │
│  │  Scheduler   │                 │  (Audio Encoder)   │   │
│  │  + Sanctum   │                 └────────┬───────────┘   │
│  │    Auth      │                          │ RTMP/HTTP PUT  │
│  └──────┬───────┘                          ▼               │
│         │ SQL queries           ┌────────────────────┐     │
│         ▼                       │     Icecast 2.4    │     │
│  ┌──────────────┐               │  (Stream Server)   │     │
│  │   MySQL DB   │               │   port :8000       │     │
│  │  (schedule,  │               └────────┬───────────┘     │
│  │   tracks,    │                        │                  │
│  │   users)     │                        │ HTTP audio stream│
│  └──────────────┘                        │                  │
│                                          │                  │
│  ┌──────────────┐                        │                  │
│  │  Supervisor  │ (keeps FFmpeg alive)   │                  │
│  └──────────────┘                        │                  │
└──────────────────────────────────────────┼─────────────────┘
                                           │
                    ┌──────────────────────┼──────────────────┐
                    │        CLIENTS       │                   │
                    │                      ▼                   │
                    │  ┌──────────────────────────────────┐   │
                    │  │        React Frontend            │   │
                    │  │  ┌──────────────┐  ┌──────────┐ │   │
                    │  │  │ RadioPlayer  │  │ Schedule │ │   │
                    │  │  │ (Audio tag   │  │   View   │ │   │
                    │  │  │ → Icecast)   │  │          │ │   │
                    │  │  └──────────────┘  └──────────┘ │   │
                    │  │         ▲                        │   │
                    │  │         │ polls /api/live-track  │   │
                    │  │         ▼                        │   │
                    │  │     Laravel REST API             │   │
                    │  └──────────────────────────────────┘   │
                    └──────────────────────────────────────────┘
```

---

## Data Flow — Step by Step

1. **Audio Storage**
   - MP3/audio files live on the server at `/var/www/radio/storage/app/music/`.
   - Laravel's `Track` model stores the filename and metadata (title, artist, duration).

2. **Laravel Scheduler** (runs every minute via `php artisan schedule:run`)
   - Queries the `playlists` and `playlist_tracks` tables to determine the **currently scheduled playlist**.
   - Computes which specific track *should* be playing at this moment based on track durations and `scheduled_at` time.
   - Passes the resolved file path to the FFmpeg process or inserts it into the stream queue.

3. **Supervisor**
   - Runs the FFmpeg worker as a managed background process.
   - Automatically restarts it within seconds if it crashes.
   - Config lives in `config/supervisor/radio-worker.conf`.

4. **FFmpeg** (`scripts/ffmpeg_stream.sh`)
   - Reads audio file(s) with `-re` flag (real-time playback rate).
   - Encodes to MP3 (128k by default).
   - Pushes the encoded stream to Icecast via HTTP PUT on the source mount.

5. **Icecast 2.4**
   - Receives the source stream and rebroadcasts it to all connected listeners.
   - Exposes the public stream at `http://<server>:8000/radio`.
   - Handles buffering, client disconnections, and queue management internally.

6. **React Frontend**
   - `RadioPlayer.jsx` sets an HTML `<audio>` element's `src` to the Icecast stream URL.
   - Every **15 seconds**, polls `GET /api/live-track` to get fresh "Now Playing" metadata.
   - `ScheduleView.jsx` polls `GET /api/upcoming-playlists` to render the timeline UI.

---

## Component Responsibilities

| Component | Responsibility | "Source of Truth" For |
| :--- | :--- | :--- |
| **Laravel** | Schedule logic, REST API, Sanctum auth, Admin role enforcement | What *should* be playing |
| **MySQL** | Persisting playlists, tracks, users, schedule | All relational data |
| **FFmpeg** | Audio decoding, encoding, and real-time streaming | Audio quality & continuity |
| **Icecast** | Client connections, buffering, stream distribution | The live stream itself |
| **Supervisor** | Process lifecycle management | FFmpeg uptime |
| **React** | Audio playback UI, metadata display, schedule view | User experience |

---

## Database Schema (High Level)

```
users
  id, name, email, password, is_admin, created_at

tracks
  id, title, artist, filename, duration_seconds, created_at

playlists
  id, name, scheduled_at, created_at

playlist_tracks  (pivot)
  id, playlist_id, track_id, order
```

---

## Authentication & Authorization

- **Sanctum Token Auth** — login returns a Bearer token stored client-side.
- **`is_admin` flag** on the `User` model — checked by the `admin` middleware on `/api/admin/*` routes.
- Public endpoints (`/api/live-track`, `/api/upcoming-playlists`) require **no auth**.
- All mutations (logout, admin stats) require a valid Sanctum token.

---

## Frontend Architecture

```
src/
├── main.jsx              # App entry, React root
├── App.jsx               # Router setup (React Router v6)
├── index.css             # Global design tokens, glassmorphism styles
├── components/
│   ├── RadioPlayer.jsx   # Audio element, visualizer, vinyl animation
│   ├── ScheduleView.jsx  # Timeline schedule display
│   └── Navbar.jsx        # Top nav, auth state display
├── pages/
│   ├── LiveRadio.jsx     # Main listener page (RadioPlayer + ScheduleView)
│   ├── Login.jsx         # Auth login form
│   ├── Register.jsx      # Auth register form
│   └── AdminDashboard.jsx # Admin-only stats view
├── context/              # React context (auth state, etc.)
└── utils/                # Shared helper functions
```

---

## Critical Constraints

- **One Stream**: All users hear identical audio. There is no per-user personalization of the stream.
- **No Silence**: FFmpeg must receive a constant flow of audio inputs, or Icecast will serve silence/disconnect clients.
- **Buffer Latency**: There is unavoidable latency (~10–30 seconds) between real-time and what clients hear. Metadata from the API is timestamped server-side so the UI can account for this drift.
- **No On-Demand**: Audio is **never** served as a static file directly. All playback goes through Icecast to preserve the live experience for late joiners.

---

## Deployment Topology

```
Internet
   │
   └── Nginx (Reverse Proxy, :80/:443)
         ├── /api/*   → Laravel (PHP-FPM, :8000 internal)
         ├── /        → React dist/ (static files)
         └── /radio   → Icecast2 (:8000 — proxied or direct)
```

> **Note:** Icecast can also be exposed directly on port 8000 if Nginx proxying introduces audio buffering issues.

---

*Last updated: March 2026*
