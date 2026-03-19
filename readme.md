
<h2 align="center">phoenyx-media</h2>

<h2 align="center">
    <img src="./.github/ulfurloyd.jpg" width="240" alt="ulfurloyd" /> <br />
</h2>

a self-hosted, automated media stack built around
**jellyfin**, **navidrome** and the **arr-stack** suite of programs.

everything routes through a **protonvpn** wireguard tunnel through
**gluetun** for a clean, secure, fully containerized setup.

## overview

`phoenyx-media` is a personal self-hosting stack built with `docker-compose`,
providing a fully automated media system that fetches, organizes, and serves movies,
shows, and music to a `jellyfin` instance.

everything runs through a VPN (via `gluetun` + `protonvpn`) to stay
private and avoid ISP interference.

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
```

## architecture

traffic from all containers passes through `gluetun`, which establishes
a secure wireguard tunnel with `protonvpn`.
the `vpn_port_forwarding` option ensures reliable torrent peer connectivity,
while `prowlarr` handles all indexer queries safely through the same route.

media downloads are processed and imported automatically by the **arrs** stack:

- `sonarr` and `radarr` monitor completed downloads,
- move them into `/media/`,
- and clean up `qbittorrent` once imported.

## local reverse proxy (dnsmasq + nginx-proxy-manager)

`dnsmasq` and `npm` work together to provide reverse
proxy access to all services that have a WebUI

this enables clean hostnames like:

```bash
http://jellyfin.phoenyx.com
http://sonarr.phoenyx.com
http://radarr.phoenyx.com
```

## directory structure

```bash
~/Dev/Docker/phoenyx-media/
│
├── docker-compose.yml
├── .env
├── /stack
│ ├── sonarr.yml
│ ├── radarr.yml
│ ├── bazarr.yml
│ ├── prowlarr.yml
│ ├── notifiarr.yml
│ ├── navidrome.yml
│ ├── nicotine.yml
│ ├── gluetun.yml
│ ├── qbittorrent.yml
│ ├── jellyfin.yml
│ ├── seerr.yml
│ ├── flaresolverr.yml
│ ├── dnsmasq.yml
│ ├── nginx.yml
└─└── homepage.yml

/media
│
├── movies/
├── shows/
└── music/
```

## configuration notes

- **VPN**: requires a ProtonVPN Plus/Unlimited account with WireGuard support.
- **Port Forwarding**: must be set up in Qbittorrent.
- **Permissions**: all containers share a common PUID/PGID via `${PUID}` and `${PUID}`.
- **Networking**: everything routes via `network_mode: "service:gluetun"`.

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
