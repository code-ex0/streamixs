Cloudflare Tunnel for streamixs.com

Goal
- Expose services (e.g., plex.streamixs.com, jellyfin.streamixs.com, etc.) without opening ports. Requires Cloudflare-managed DNS for streamixs.com.

Steps
1) Install `cloudflared` locally and authenticate to create a tunnel:
```
cloudflared tunnel login
cloudflared tunnel create media-tunnel
cloudflared tunnel token media-tunnel
```
- Copy the token into `.env` as `CF_TUNNEL_TOKEN=...`

2) DNS in Cloudflare
- In the tunnel UI, add public hostnames that map to local services:
  - `plex.streamixs.com` -> http://plex:32400
  - `jellyfin.streamixs.com` -> http://jellyfin:8096
  - `prowlarr.streamixs.com` -> http://prowlarr:9696
  - `radarr.streamixs.com` -> http://radarr:7878
  - `sonarr.streamixs.com` -> http://sonarr:8989
  - `qbittorrent.streamixs.com` -> http://qbittorrent:8080
  - `bazarr.streamixs.com` -> http://bazarr:6767
  - `overseerr.streamixs.com` -> http://overseerr:5055
  - `tautulli.streamixs.com` -> http://tautulli:8181
  - `tdarr.streamixs.com` -> http://tdarr:8265

- Alternatively, use a config file mounted at `/home/nonroot/.cloudflared/config.yml` to define ingresses (advanced usage).

3) Start the tunnel with compose
```
docker compose up -d cloudflared
```

4) TLS/Access (optional)
- Enforce Cloudflare Access policies per subdomain (zero-trust) to restrict usage.
- For Plex, you may need to allow websocket and long-lived connections.

Notes
- The compose uses `TUNNEL_TOKEN` mode, so no volumes/credentials are required. You can switch to the credentials-file mode if you prefer.`
- All backend hostnames refer to container names on the same Docker network (`media_net`).
