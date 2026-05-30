
<h2 align="center">phoenyx-media</h2>

<h2 align="center">
    <img src="./.github/ulfurloyd.jpg" width="240" alt="ulfurloyd" /> <br />
</h2>

> [!NOTE]
> ongoing migration from docker compose to k3s

a self-hosted, automated media stack built around
**jellyfin**, **navidrome** and the **arr-stack** suite of programs.

everything routes through a **protonvpn** wireguard tunnel through
**gluetun** for a clean, secure, fully containerized setup.

## overview

`phoenyx-media` is a personal self-hosting stack built with `docker-compose`,
providing a fully automated media system that fetches, organizes, and serves movies,
shows, and music to a `jellyfin` instance.

sensitive data runs through a VPN (via `gluetun` + `protonvpn`) to stay
private and avoid ISP interference.

traffic is routed through the network using a `duckdns` domain.
`nginx-proxy-manager` controls TLS certificates using the `duckdns` domain.

## stack components

```
| service          | role                                                      |
|------------------|-----------------------------------------------------------|
| **jellyfin**     | media server (movies, shows, music)                       |
| **seerr**        | request manager for jellyfin                              |
| **prowlarr**     | indexer management for the arrs                           |
| **sonarr**       | tv show automation                                        |
| **radarr**       | movie automation                                          |
| **bazarr**       | subtitle acquisition automation                           |
| **nicotine+**    | music acquisition                                         |
| **navidrome+**   | music server                                              |
| **qbittorrent**  | download client (routed through gluetun)                  |
| **gluetun**      | VPN gateway using ProtonVPN (WireGuard + port forwarding) |
| **flaresolverr** | cloudflare bypass proxy for indexers                      |
| **homepage**     | lightweight dashboard for quick access to stack services  |
| **dnsmasq**      | local DNS resolution for media services                   |
| **npm**          | local reverse proxy                                       |
| **diun**         | tracking and notifying image updates                      |
```

## architecture

the stack is built around a **two-network model** that cleanly separates
internal service communication from vpn-routed traffic:

```plaintext
media_net -> internal service communication
vpn_net   -> outbound traffic through vpn (via gluetun)
```

### vpn routing (gluetun)
`gluetun` acts as a dedicated vpn gateway using wireguard with protonvpn.

only services that require anonymity or external scraping are routed through it:

- `qbittorrent` (torrent traffic)
- `prowlarr` (indexer queries)
- `flaresolverr` (anti-bot bypass)
- `nicotine` (music)

these containers share `gluetun`'s network namespace (`network_mode: service:gluetun`), ensuring all their trafficexits through the vpn tunnel.

`gluetun` is also attached to `media_net`, allowing internal services to access
vpn-routed services via:

```plaintext
http://gluetun:<port>
```

### internal service mesh (media_net)

all core applications run on `media_net`, forming a clean internal network:

- `sonarr`, `radarr`, `bazarr`
- `seerr`
- `jellyfin`
- `navidrome`
- `homepage`
- `notifiarr`

services communicate using docker dns:

```plaintext
http://sonarr:8989
http://radarr:7878
http://gluetun:9696  # prowlarr via vpn
http://gluetun:8080  # qbittorent via vpn
```

this avoids localhost/ip-based routing and ensures reliable inter-service
communication.

### download + media workflow

the arr stack automates the full pipeline:

- prowlarr aggregates indexers (via vpn)
- sonarr / radarr handle search and download requests
- qbittorrent downloads content (via vpn)
- completed downloads are:
    - imported into /media
    - organized automatically
    - removed from the download client

### ingress (dnsmasq + nginx proxy manager)

all user access flows through a single ingress layer:

- dnsmasq resolves *.phoenyxlab.duckdns.org → local server ip
- Nginx Proxy Manager handles routing + https termination

this enables clean, consistent endpoints:

```bash
https://jellyfin.phoenyxlab.duckdns.org
https://sonarr.phoenyxlab.duckdns.org
https://radarr.phoenyxlab.duckdns.org
https://homepage.phoenyxlab.duckdns.org
```

no services are exposed directly via host ports (except ingress and vpn endpoints)

### update monitoring

Diun monitors container images and sends notifications (via discord webhook) when updates are available.

only selected containers are tracked using labels:

```yaml
labels:
  - diun.enable=true
```

## configuration notes

- **VPN**: requires a ProtonVPN Plus/Unlimited account with WireGuard support.
- **Port Forwarding**: must be set up in Qbittorrent.
- **Permissions**: all containers share a common PUID/PGID via `${PUID}` and `${PUID}`.

## automation features

- **auto-fetches** requested content via `sonarr` and `radarr`
- **cleans up** finished downloads after import into `/media`
- **centralized indexer management** through `prowlarr`
- **VPN-secured downloads** with Gluetun + ProtonVPN
- **Cloudflare bypass** through `flaresolverr`
- **Seerr integration** for user requests
- **Homepage dashboard** for quick access to all services

## environment variables

`.env` example:

```bash
PUID=956
PGID=947
TZ=Asia/Kolkata
WIREGUARD_PRIVATE_KEY=your_wireguard_private_key_here
CONFIG_BASE_PATH=/srv/docker/phoenyx-media
DOWNLOADS_PATH=/var/lib/qbittorrent/Downloads
MOVIES_LIBRARY_PATH=/media/movies
SHOWS_LIBRARY_PATH=/media/shows
MUSIC_LIBRARY_PATH=/media/music
```
