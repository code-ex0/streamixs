Media stack with Plex, Jellyfin, Prowlarr, Radarr, Sonarr, qBittorrent, Bazarr, Overseerr, Tautulli, FlareSolverr, and Tdarr.

Prereqs
- Docker Desktop for macOS

Setup
1) Adjust `.env` if desired (paths, timezone, PUID/PGID).
2) Start the stack:

docker compose up -d

3) Open services:
- Plex: http://localhost:32400/web
- Jellyfin: http://localhost:8096
- Prowlarr: http://localhost:9696
- qBittorrent: http://localhost:8080
- Radarr: http://localhost:7878
- Sonarr: http://localhost:8989
- Bazarr: http://localhost:6767
- Overseerr: http://localhost:5055
- Tautulli: http://localhost:8181
- FlareSolverr: http://localhost:8191
- Tdarr: http://localhost:8265

Paths
- Configs: `${CONFIG_ROOT}`
- Media library: `${MEDIA_ROOT}` (subfolders: `movies`, `tv`)
- Downloads: `${DOWNLOADS_ROOT}`
- Transcode/temp: `${TRANSCODE_ROOT}`

Notes
- On macOS, hardware transcoding inside containers is limited. Software transcoding works by default.
- Make sure Radarr/Sonarr and qBittorrent use the same `downloads` path so completed files can be imported.
- Tdarr ports: 8265 (Web), 8266 (Server), 8267 (Node).

Common commands

# Start/Stop
# docker compose up -d
# docker compose down

# Logs
# docker compose logs -f radarr

# Update images
# docker compose pull && docker compose up -d
