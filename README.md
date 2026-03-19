# 📻 Live Radio Streaming System

A production-ready, scheduling-first live radio platform built with **Laravel** (Backend), **React** (Frontend), **Icecast** (Streaming), and **FFmpeg**.

> All listeners tune in to the **same broadcast simultaneously** — a true radio experience, not on-demand playback.

---

## ✨ Features

| Feature | Description |
|---|---|
| 🎙️ **Live Audio Broadcasting** | Icecast + FFmpeg pipeline for 24/7 continuous streaming |
| 🗓️ **Strict Scheduling** | Laravel Scheduler drives the playlist based on time-of-day |
| 🔴 **Live Metadata** | Real-time "Now Playing" updates via API polling |
| 🔐 **Auth & Roles** | Sanctum-based login/register with Admin & User roles |
| 🎨 **Modern UI** | Cyberpunk/Glassmorphism aesthetic with audio visualizer, vinyl animations, and timeline schedule view |
| 🔄 **Auto-Recovery** | Supervisor keeps the FFmpeg stream alive and restarts it on failure |
| 📱 **Responsive Design** | Mobile-ready layout |

---

## 🛠 Tech Stack

- **Backend**: Laravel 10+ (PHP 8.2), Laravel Sanctum
- **Frontend**: React.js (Vite), React Router v6, CSS3 Variables / Glassmorphism
- **Streaming**: Icecast 2.4, FFmpeg 6.0
- **Database**: MySQL / MariaDB
- **Process Manager**: Supervisor
- **OS**: Linux (Ubuntu/Debian recommended)

---

## 📁 Project Structure

```
/
├── backend/          # Laravel API, Scheduler, Auth, Controllers
├── frontend/         # React Audio Player, Schedule UI, Auth Pages
├── config/           # Icecast & Supervisor configuration files
├── scripts/          # Helper shell scripts (ffmpeg_stream.sh, etc.)
└── docs/             # Architecture, Failure Handling & External API docs
```

---

## 🚀 Quick Start

### Prerequisites
Ensure the following are installed on your server:
```
ffmpeg  icecast2  php (8.2+)  composer  node (18+)  npm  supervisor
```

### 1. Backend Setup
```bash
cd backend
composer install
cp .env.example .env
# Edit .env with your DB credentials and Icecast config
php artisan key:generate
php artisan migrate
php artisan db:seed   # Optional: seeds admin user & sample tracks
```

### 2. Frontend Setup
```bash
cd frontend
npm install
npm run dev     # Development
npm run build   # Production build (outputs to dist/)
```

### 3. Start the Stream
Enable the Supervisor config to launch the background FFmpeg worker:
```bash
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start radio-worker
```

---

## 🔌 API Endpoints

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| `GET` | `/api/live-track` | Public | Returns currently playing track metadata |
| `GET` | `/api/upcoming-playlists` | Public | Returns the upcoming schedule |
| `POST` | `/api/register` | Public | Register a new user |
| `POST` | `/api/login` | Public | Login and receive Sanctum token |
| `POST` | `/api/logout` | 🔒 Sanctum | Logout current user |
| `GET` | `/api/user` | 🔒 Sanctum | Get authenticated user info |
| `GET` | `/api/admin/stats` | 🔒 Admin | Admin dashboard stats |

---

## 🖥 Frontend Routes

| Path | Component | Access |
|---|---|---|
| `/` | `LiveRadio` | Public |
| `/login` | `Login` | Public |
| `/register` | `Register` | Public |
| `/admin` | `AdminDashboard` | Admin Only |

---

## ⚙️ Environment Variables

After copying `.env.example` to `.env`, configure at minimum:

```dotenv
APP_NAME="Live Radio"
APP_URL=http://your-domain.com

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=live_radio
DB_USERNAME=root
DB_PASSWORD=your_password

# Icecast stream source password
ICECAST_SOURCE_PASSWORD=your_icecast_password
```

---

## 📖 Documentation

| Document | Description |
|---|---|
| [ARCHITECTURE.md](docs/ARCHITECTURE.md) | System design and component interactions |
| [FAILURE_HANDLING.md](docs/FAILURE_HANDLING.md) | How the system handles stream drops and auto-recovers |
| [EXTERNAL_APIS.md](docs/EXTERNAL_APIS.md) | Details on external integrations |

---

## 🤝 Contributing

1. Fork the project
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

---

## 📄 License

MIT — see [LICENSE](LICENSE) for details.
