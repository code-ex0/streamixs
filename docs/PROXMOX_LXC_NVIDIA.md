Proxmox LXC + NVIDIA GPU passthrough for Plex/Jellyfin/Tdarr

Host (Proxmox node)
1) Install NVIDIA driver on the host and reboot:
   - Use Proxmox wiki/guide to install proprietary NVIDIA driver
   - Verify: `nvidia-smi`

2) Create a privileged LXC (Ubuntu 22.04/24.04 recommended) with features:
   - `features: keyctl=1,nesting=1` (and optionally `fuse=1`)

3) Edit `/etc/pve/lxc/<CTID>.conf` and add (adjust if some files are missing):
```
features: keyctl=1,nesting=1
lxc.apparmor.profile: unconfined
lxc.cgroup2.devices.allow: c 195:* rwm
lxc.cgroup2.devices.allow: c 511:* rwm
lxc.mount.entry: /dev/nvidia0 dev/nvidia0 none bind,create=file,optional
lxc.mount.entry: /dev/nvidiactl dev/nvidiactl none bind,create=file,optional
lxc.mount.entry: /dev/nvidia-modeset dev/nvidia-modeset none bind,create=file,optional
lxc.mount.entry: /dev/nvidia-uvm dev/nvidia-uvm none bind,create=file,optional
lxc.mount.entry: /dev/nvidia-uvm-tools dev/nvidia-uvm-tools none bind,create=file,optional
```
- Restart the CT: `pct stop <CTID> && pct start <CTID>`
- Inside the CT, confirm: `ls -l /dev/nvidia*`

Container (inside the LXC)
1) Install Docker Engine and NVIDIA Container Toolkit:
```
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo $VERSION_CODENAME) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update && sudo apt-get install -y docker-ce docker-ce-cli containerd.io

# NVIDIA toolkit
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit.gpg
. /etc/os-release
curl -s -L https://nvidia.github.io/libnvidia-container/$ID/$VERSION_ID/libnvidia-container.list | \
  sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit.gpg] https://#g' | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```

2) Test GPU in Docker:
```
sudo docker run --rm --gpus all nvidia/cuda:12.2.0-base-ubuntu22.04 nvidia-smi
```

3) Prepare volumes and env inside the CT (adjust paths):
```
sudo mkdir -p /srv/plex/{config/{plex,jellyfin,prowlarr,radarr,sonarr,tdarr/server,tdarr/configs,tdarr/logs,qbittorrent,bazarr,overseerr,tautulli},media/{movies,tv},downloads,transcode}
```

4) Clone/copy this repo folder into the CT and set `.env` (or use `.env.proxmox`):
- Set `CONFIG_ROOT=/srv/plex/config`, `MEDIA_ROOT=/srv/plex/media`, `DOWNLOADS_ROOT=/srv/plex/downloads`, `TRANSCODE_ROOT=/srv/plex/transcode`
- Set `PUID`/`PGID` to the user that owns those paths (often 1000:1000)

5) Enable NVIDIA for services
- Option A (recommended): Keep compose as-is and set Docker default runtime to `nvidia` (already done by `nvidia-ctk`).
- Option B: Uncomment the `runtime: nvidia` and `devices:` blocks in `docker-compose.yml` for `plex`, `jellyfin`, `tdarr`, and `tdarr-node` if needed.

6) Bring up the stack:
```
cd /path/to/plex
sudo --preserve-env=TZ,PUID,PGID,UMASK,CONFIG_ROOT,MEDIA_ROOT,DOWNLOADS_ROOT,TRANSCODE_ROOT \
  docker compose up -d
```

Notes
- Tdarr server UI: http://<CT-IP>:8265
- Plex hardware transcoding requires Plex Pass.
- If you use Intel iGPU instead, pass `/dev/dri` into the LXC and map it into containers (`devices: - /dev/dri:/dev/dri`).
