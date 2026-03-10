# Live Radio Streaming System

A production-ready, scheduling-first live radio system built with **Laravel** (Backend), **React** (Frontend), **Icecast** (Streaming), and **FFmpeg**.

![UI Preview](https://via.placeholder.com/800x400?text=ECHO+WAVE+RADIO+UI)

## Overview
This system enables a 24/7 radio broadcast where all listeners hear the same audio simultaneously. It uses a server-side scheduler to strictly manage what tracks play at specific times, ensuring a unified "broadcast" experience rather than on-demand playback.

## Features
-   **Audio Broadcasting**: Uses Icecast + FFmpeg for continuous streaming.
-   **Strict Scheduling**: Laravel Scheduler determines the playlist based on time-of-day.
-   **Live Metadata**: Real-time "Now Playing" updates via API polling.
-   **Authentication & Roles**: Secure Login and Registration with role-based access control (Admin/User).
-   **Modern UI**: "Cyberpunk/Glassmorphism" aesthetic with:
    -   Premium Glassmorphism Login and Registration pages.
    -   Real-time audio visualizer animations.
    -   Vinyl record rotation effects.
    -   Vertical timeline schedule view.
    -   Responsive mobile-ready layout.
-   **Auto-Recovery**: Supervisor keeps the stream alive and restarts it on failure.

## Tech Stack
-   **Backend**: Laravel 10+ (PHP 8.2)
-   **Frontend**: React.js (Vite/CRA), CSS3 Variables, Glassmorphism
-   **Streaming**: Icecast 2.4, FFmpeg 6.0
-   **Database**: MySQL / MariaDB
-   **OS**: Linux (Ubuntu/Debian recommended)

## Project Structure
-   `/backend` - Laravel API & Scheduler logic.
-   `/frontend` - React Audio Player & Schedule UI.
-   `/config` - Server configurations (Icecast, Supervisor).
-   `/scripts` - Helper scripts for streaming.
-   `/docs` - Architecture validation and failure handling docs.

## Quick Start
See [walkthrough.md](walkthrough.md) for detailed deployment instructions.

### 1. Requirements
Ensure `ffmpeg`, `icecast2`, `php`, `composer`, `node`, and `supervisor` are installed.

### 2. Run Backend
```bash
cd backend
composer install
cp .env.example .env
# Configure your MySQL database credentials in the .env file
php artisan key:generate
php artisan migrate
```

### 3. Run Frontend
```bash
cd frontend
npm install
npm start
```

### 4. Start Streaming
Enable the Supervisor config provided in `config/radio-worker.conf` to start the background worker.

## Documentation
For deeper technical insights, please refer to the documents in the `/docs` directory:
- [Architecture Details](docs/ARCHITECTURE.md) - System design and component interactions.
- [Failure Handling](docs/FAILURE_HANDLING.md) - How the system handles stream drops and auto-recovers.
- [External APIs](docs/EXTERNAL_APIS.md) - Details on external integrations.

## Environment Variables
The backend relies on several environment variables. After copying `.env.example` to `.env`, ensure you configure at least the following:
```dotenv
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=live_radio
DB_USERNAME=root
DB_PASSWORD=your_password
```

## Database Seeding
To populate the database with initial data (like an admin user or sample tracks), run:
```bash
cd backend
php artisan db:seed
```

## Contributing
1. Fork the project.
2. Create your feature branch (`git checkout -b feature/AmazingFeature`).
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`).
4. Push to the branch (`git push origin feature/AmazingFeature`).
5. Open a Pull Request.

## License
MIT
