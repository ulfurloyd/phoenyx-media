
<h2 align="center">phoenyxlab</h2>

<h2 align="center">
    <img src="./.github/ulfurloyd.jpg" width="240" alt="ulfurloyd" /> <br />
</h2>

[![.github/workflows/validate-images.yaml](https://github.com/ulfurloyd/phoenyxlab/actions/workflows/validate-images.yaml/badge.svg?branch=main)](https://github.com/ulfurloyd/phoenyxlab/actions/workflows/validate-images.yaml)

self-hosted home network server running via **kubernetes**
test

## overview

`phoenyxlab` is a personal self-hosting stack running in `kubernetes`,
providing media, monitoring, DNS, and infrastructure services for my
home network.

secure HTTPS routing for all services from a domain I own (`phoenyxlab.xyz`).

complete reverse-proxy support via `traefik`.

container images are locked to the latest stable tag.
`renovate` keeps track of image updates and opens a PR whenever a
new image is available.

`sops` and `age` encrypt `secrets/*`


## network

```
| hostname           | service        | namespace   | port  | description    |
|--------------------+----------------+-------------+-------+----------------|
| **homarr**         | homarr         | infra       | 7575  | Dashboard      |
| **notifiarr**      | notifiarr      | infra       | 5454  | Notifications  |
| **pi-hole**        | pihole         | infra       | 80    | DNS            |
| **jellyfin**       | phoenyxfin     | media       | 8096  | Media Server   |
| **navidrome**      | navidrome      | media       | 4533  | Music Server   |
| **radarr**         | radarr         | media       | 7878  | Movies         |
| **seerr**          | seerr          | media       | 5055  | Requests       |
| **sonarr**         | sonarr         | media       | 8989  | Shows          |
| **soulsync**       | soulsync       | media       | 8008  | Music Syncing  |
| **jackett**        | jackett        | downloads   | 9117  | Indexers       |
| **prowlarr**       | prowlarr       | downloads   | 9696  | Indexers       |
| **qbittorrent**    | qbittorrent    | downloads   | 8080  | Torrents       |
| **slskd**          | slskd          | downloads   | 5030  | Soulseek       |
| **cadvisor**       | cadvisor       | monitoring  | 8080  | Containers     |
| **grafana**        | grafana        | monitoring  | 3000  | Dashboards     |
| **node-exporter**  | node-exporter  | monitoring  | 9100  | Host metrics   |
| **prometheus**     | prometheus     | monitoring  | 9090  | Metrics        |
```
...all accessible at `https://<service>.phoenyxlab.xyz`

## kubernetes

while the choice of using **kubernetes** started as a way to learn the tech,
it's quickly become a beloved set of tools that runs my entire infra.

### core infrastructure

- **traefik** for ingress and HTTPS routing
- **pi-hole** for network-wide DNS and local service discovery
- **prometheus** + **grafana** for monitoring
- **homarr** as the primary service dashboard

### monitoring

- **prometheus** for metrics collection
- **grafana** for dashboards
- **cadvisor** for container metrics
- **node exporter** for host metrics

### namespaces

namespaces provide logical partitioning between pods.
services in different namespaces can find each other by
looking for `<service>.<namespace>:<port>`. for example,
`soulsync` finds `slskd` at `slskd.downloads:5030`

- media
- downloads
- infra
- monitoring

### continuous deployment

`FluxCD` provides continuous deployment features, watching
the repo for changes and applying them locally.

## download + media workflow

the arr stack automates the full pipeline:

- jackett aggregates indexers
- sonarr / radarr handle search and download requests
- qbittorrent downloads content
- completed downloads are:
    - imported into /media
    - organized automatically
    - removed from the download client

## automation features

- **auto-fetches** requested content via `sonarr`, `radarr` and `soulsync`
- **cleans up** finished downloads after import into `/media`
- **centralized indexer management** through `prowlarr`
- **VPN-secured downloads** with Gluetun + ProtonVPN
- **Seerr integration** for user requests
- **Homarr dashboard** for quick access to all services


## bootstrap
- decrypt and apply secrets with sops/age
- apply namespaces
- apply tlsstore
- apply pihole-redirect middleware
