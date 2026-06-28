
<h1 align="center">phoenyxlab</h1>

<h2 align="center">
    <img src="./.github/ulfurloyd.jpg" width="240" alt="ulfurloyd" /> <br />
</h2>

[![.github/workflows/validate-images.yaml](https://github.com/ulfurloyd/phoenyxlab/actions/workflows/validate-images.yaml/badge.svg?branch=main)](https://github.com/ulfurloyd/phoenyxlab/actions/workflows/validate-images.yaml) [![Validate Kubernetes](https://github.com/ulfurloyd/phoenyxlab/actions/workflows/validate-kubernetes.yaml/badge.svg)](https://github.com/ulfurloyd/phoenyxlab/actions/workflows/validate-kubernetes.yaml)

self-hosted home network server running via **kubernetes**


## overview

`phoenyxlab` is a personal self-hosting stack running in `kubernetes`,
providing media, monitoring, DNS, and infrastructure services for my
home network.

secure HTTPS routing for all services from a domain I own (`phoenyxlab.xyz`).

services are routed through private LAN DNS and remote access is handled via
Tailscale; the cluster is not intended to be publicly reachable.

reverse-proxy and ingress management via `traefik`.

container images and Helm charts are pinned to explicit versions where practical.
`renovate` monitors upstream releases and opens a PR whenever a
new version is available.

`sops` and `age` encrypt Kubernetes Secrets and selected sensitive values in
manifests.


## network

```
+--------------------+----------------+-------------+-------+----------------+
| hostname           | service        | namespace   | port  | description    |
|--------------------+----------------+-------------+-------+----------------|
| traefik            | api@internal   | kube-system | 443   | Proxy dashboard|
| homarr             | homarr         | infra       | 7575  | Dashboard      |
| homepage           | homepage       | infra       | 3000  | Dashboard      |
| notifiarr          | notifiarr      | infra       | 5454  | Notifications  |
| pi-hole            | pihole         | infra       | 80    | DNS            |
| vaultwarden        | vaultwarden    | infra       | 80    | Passwords      |
| freshrss           | freshrss       | media       | 80    | RSS Reader     |
| jellyfin           | phoenyxfin     | media       | 8096  | Media Server   |
| lidarr             | lidarr         | media       | 8686  | Music          |
| navidrome          | navidrome      | media       | 4533  | Music Server   |
| pinepods           | pinepods       | media       | 8040  | Podcasts       |
| radarr             | radarr         | media       | 7878  | Movies         |
| seerr              | seerr          | media       | 5055  | Requests       |
| sonarr             | sonarr         | media       | 8989  | Shows          |
| soulsync           | soulsync       | media       | 8008  | Music Syncing  |
| jackett            | jackett        | downloads   | 9117  | Indexers       |
| prowlarr           | prowlarr       | downloads   | 9696  | Indexers       |
| qbittorrent        | qbittorrent    | downloads   | 8080  | Torrents       |
| slskd              | slskd          | downloads   | 5030  | Soulseek       |
| alertmanager       | alertmanager   | monitoring  | 9093  | Alerts         |
| cadvisor           | cadvisor       | monitoring  | 8080  | Containers     |
| glances            | glances        | monitoring  | 61208 | System Stats   |
| grafana            | grafana        | monitoring  | 3000  | Dashboards     |
| node-exporter      | node-exporter  | monitoring  | 9100  | Host metrics   |
| prometheus         | prometheus     | monitoring  | 9090  | Metrics        |
| uptime             | uptime-kuma    | external    | 443   | Uptime checks  |
+--------------------+----------------+-------------+-------+----------------+
```
exposed web services are accessible at `https://<hostname>.phoenyxlab.xyz`

## kubernetes

while the choice of using **kubernetes** started as a way to learn the tech,
it's quickly become a beloved set of tools that runs my entire infra.

### cluster architecture

```
Git
 ↓
FluxCD
 ↓
Kubernetes
 ├─ media
 ├─ downloads
 ├─ infra
 ├─ monitoring
 ├─ cert-manager
 └─ backup
```

### core infrastructure

- **traefik** for ingress and HTTPS routing
- **cert-manager** for automatic TLS certificate issuance and renewal
- **pi-hole** for network-wide DNS and local service discovery
- **prometheus** + **grafana** for monitoring
- **uptime-kuma** runs externally on OCI for out-of-cluster monitoring
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
- cert-manager
- backup

### persistent storage

app state is primarily stored in **Persistent Volume Claims**, while
declarative configuration lives in Git as manifests, ConfigMaps, and
SOPS-encrypted Secrets.

PVC-backed app data is backed up monthly to Cloudflare R2 using **Restic**.

### validation

GitHub Actions validate Kubernetes manifests with `kubeconform` and verify
changed container image tags before deployment.

`renovate` tracks Kubernetes image tags and Flux HelmRelease chart versions
across cluster and application manifests.

### continuous deployment

`FluxCD` watches the repo for changes and reconciles
cluster state automatically.

deployment events and failures are sent to Discord via
Flux notifications.

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

## uptime monitoring

- **uptime-kuma** is hosted externally on an Oracle Cloud Infrastructure
(OCI) virtual machine.
- it's accessible at `uptime.phoenyxlab.xyz`, secured behind 2FA.
- the VM is configured as a tailscale node so cluster services can
access it via a private, internal connection.

## bootstrap

1. install k3s
2. install FluxCD
3. restore the SOPS age key
4. bootstrap Flux against the repo
5. Flux reconciles cluster state

## disaster recovery

- monthly Restic backups run via a Kubernetes CronJob
- backups are encrypted client-side before upload
- snapshots are stored in a Cloudflare R2 bucket
- monthly retention policy keeps 12 snapshots
- `restic-r2-secret` contains repo credentials
