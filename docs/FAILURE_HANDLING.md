# 🚨 Common Failure Cases & Handling

A Live Radio system has unique failure modes compared to a standard web application because it relies on continuous, real-time data streaming (Icecast + FFmpeg) combined with scheduled metadata (Laravel).

Here is how the system is designed to handle various failures, and how to recover if things go wrong.

---

## 1. Dead Air (Silence)
**Symptom**: Listeners are connected to the stream, the player shows it's playing, but they hear nothing.
**Cause**: The FFmpeg process died, hung, or finished its current audio file and didn't receive the next one from the Scheduler.
**Recovery & Mitigation**:
- **Primary:** Supervisor is configured to auto-restart the `radio-worker` process if it crashes.
- **Secondary (Fail-safe):** A Laravel Scheduled Task (`php artisan schedule:run`) checks the Icecast admin API every minute. If no source is connected, it forces a restart of the worker.
- **Fallback Audio:** The Icecast configuration (`icecast.xml`) uses `<fallback-mount>` to seamlessly transition listeners to a looping "We'll be right back" track if the main FFmpeg source disconnects, preventing clients from dropping the connection.

## 2. Stream Stuttering / Frequent Buffering
**Symptom**: The audio cuts in and out continuously for listeners.
**Cause**: Insufficient server network bandwidth, high CPU usage on the FFmpeg encoder, or poor client connection.
**Fixes**:
- **Reduce Bitrate:** Lower the FFmpeg encoding bitrate from `128k` to `96k` or `64k` in `scripts/ffmpeg_stream.sh`.
- **System Resources:** Check Server CPU. FFmpeg transcoding is CPU intensive. Scale up the server if CPU is consistently > 90%.
- **Client Buffering:** Increase the Icecast `<queue-size>` or `<burst-size>` in `icecast.xml` to send more buffer data to clients upon initial connection, smoothing out minor network hiccups.

## 3. Metadata Desync (Wrong Song Showing)
**Symptom**: The "Now Playing" UI shows the wrong track, or changes a few seconds before/after the audio actually changes.
**Cause**: The naturally occurring latency between the server generating the audio stream and the client's `<audio>` tag buffering and playing it.
**Fixes**:
- **Timestamp Compensation (Current):** The Laravel API provides a `played_at` timestamp. The React client can use this to anticipate or delay UI updates slightly, though exact sync is difficult without stream-embedded metadata.
- **ICY Metadata (Future Enhancement):** Instead of React polling for metadata, FFmpeg can inject the track name directly into the Icecast stream (ICY protocol). The client audio player then extracts this metadata exactly as the corresponding audio frames are played.

## 4. Late Joiner Mechanics
**Symptom**: A user tunes in halfway through a scheduled segment.
**Handling**:
- **Live Nature:** This is the intended behavior. Unlike an on-demand Spotify track, the user joins the *live broadcast*.
- **Implementation:** FFmpeg is invoked with the `-re` flag (Read input at native frame rate). This ensures that even if FFmpeg processes a 5-minute song, it takes exactly 5 minutes to stream it. A user joining at minute 3 connects to Icecast and receives only the remaining 2 minutes. The backend must *never* serve the static MP3 file directly to the frontend.

## 5. Corrupted or Missing Files
**Symptom**: The system schedule expects a track, but the underlying file (`.mp3`) was deleted or corrupted.
**Cause**: Manual deletion, failed upload, or storage failure.
**Handling**:
- **Pre-check:** Before passing a file to FFmpeg, the Laravel Scheduler performs a `file_exists()` and readability check.
- **Skip Logic:** If the file is missing or unreadable, the Scheduler immediately logs an error and skips to the *next* scheduled track to minimize dead air, notifying the admin via Laravel logs.

## 6. Database / API Outage
**Symptom**: Metadata isn't updating, or the frontend can't load the schedule.
**Cause**: MySQL is down or Laravel API is unresponsive.
**Handling**:
- **Stream Resilience:** The Icecast + FFmpeg pipeline can often survive a brief API outage. If FFmpeg was given a multi-hour playlist script, it will continue broadcasting the audio schedule even if the frontend UI fails to load metadata.
- **UI Graceful Degradation:** The React frontend caches the last known schedule and displays a "Live (Metadata Unavailable)" state rather than crashing the audio player if `GET /api/live-track` fails.

## 7. Supervisor / Process Manager Failure
**Symptom**: Everything stops. `radio-worker` won't start.
**Fix**:
- Check Supervisor logs: `tail -f /var/log/supervisor/radio-worker.log`
- Ensure the `scripts/ffmpeg_stream.sh` file has execute permissions: `chmod +x scripts/ffmpeg_stream.sh`.
- Ensure the paths defined in the supervisor config are absolute and correct for the production server environment.
