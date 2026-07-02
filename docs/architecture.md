
# service reference

## access patterns

services are generally exposed in three ways:

```
+--------------+---------------------------------+----------------------------+
| type         | example                         | usage                      |
+--------------+---------------------------------+----------------------------+
| Ingress      | https://jellyfin.phoenyxlab.xyz | user-facing apps           |
| ClusterIP    | freshrss-postgres.media:5432    | internal svc communication |
| LoadBalancer | pihole-dns                      | LAN services               |
+--------------+---------------------------------+----------------------------+
```

---

# downloads

## gluetun

```
+----------------+------------------------------------------------------------+
| Property       | Value                                                      |
+----------------+------------------------------------------------------------+
| Namespace      | downloads                                                  |
| Service        | n/a (config/secret only)                                   |
| Container Port | n/a                                                        |
| Service Type   | n/a                                                        |
| External URL   | n/a                                                        |
| Dependencies   | ProtonVPN (WireGuard)                                      |
| Storage        | none                                                       |
+----------------+------------------------------------------------------------+
```

WireGuard VPN client (ProtonVPN) via ConfigMap + SOPS-encrypted Secret. Used as an initContainer by prowlarr and qbittorrent. Enables port forwarding.

## jackett

```
+----------------+------------------------------------------------------------+
| Property       | Value                                                      |
+----------------+------------------------------------------------------------+
| Namespace      | downloads                                                  |
| Service        | jackett                                                    |
| Container Port | 9117                                                       |
| Service Type   | ClusterIP                                                  |
| External URL   | https://jackett.phoenyxlab.xyz                             |
| Dependencies   | gluetun (indirect via tun), flaresolverr (initContainer)   |
| Storage        | jackett-config PVC (1Gi), /dev/net/tun                     |
+----------------+------------------------------------------------------------+
```

Init container runs flaresolverr on port 8191.

## prowlarr

```
+----------------+------------------------------------------------------------+
| Property       | Value                                                      |
+----------------+------------------------------------------------------------+
| Namespace      | downloads                                                  |
| Service        | prowlarr                                                   |
| Container Port | 9696                                                       |
| Service Type   | ClusterIP                                                  |
| External URL   | https://prowlarr.phoenyxlab.xyz                            |
| Dependencies   | gluetun (initContainer)                                    |
| Storage        | prowlarr-config PVC (5Gi), /dev/net/tun                    |
+----------------+------------------------------------------------------------+
```

## qbittorrent

```
+----------------+------------------------------------------------------------+
| Property       | Value                                                      |
+----------------+------------------------------------------------------------+
| Namespace      | downloads                                                  |
| Service        | qbittorrent                                                |
| Container Port | 8080                                                       |
| Service Type   | ClusterIP                                                  |
| External URL   | https://qbittorrent.phoenyxlab.xyz                         |
| Dependencies   | gluetun (initContainer, port forwarding)                   |
| Storage        | qbittorrent-config PVC (2Gi), /downloads/qbittorrent       |
+----------------+------------------------------------------------------------+
```

## slskd

```
+----------------+------------------------------------------------------------+
| Property       | Value                                                      |
+----------------+------------------------------------------------------------+
| Namespace      | downloads                                                  |
| Service        | slskd                                                      |
| Container Port | 5030 (web), 50300 (P2P)                                    |
| Service Type   | ClusterIP (5030 only)                                      |
| External URL   | https://slskd.phoenyxlab.xyz                               |
| Dependencies   | none                                                       |
| Storage        | slskd-config PVC (2Gi), /downloads/slskd/completed,        |
|                | /downloads/slskd/incomplete                                |
+----------------+------------------------------------------------------------+
```

Soulseek client. P2P port 50300 is not exposed via Service.

---

# media

## freshrss

```
+----------------+------------------------------------------------------------+
| Property       | Value                                                      |
+----------------+------------------------------------------------------------+
| Namespace      | media                                                      |
| Service        | freshrss                                                   |
| Container Port | 80                                                         |
| Service Type   | ClusterIP                                                  |
| External URL   | https://freshrss.phoenyxlab.xyz                            |
| Dependencies   | freshrss-postgres                                          |
| Storage        | freshrss-data PVC (5Gi)                                    |
+----------------+------------------------------------------------------------+
```

### sub-services

**freshrss-postgres**
```
+----------------+------------------------------------------------------------+
| Property       | Value                                                      |
+----------------+------------------------------------------------------------+
| Namespace      | media                                                      |
| Service        | freshrss-postgres                                          |
| Container Port | 5432                                                       |
| Service Type   | ClusterIP                                                  |
| Dependencies   | none                                                       |
| Storage        | freshrss-postgres PVC (5Gi)                                |
+----------------+------------------------------------------------------------+
```

## jellyfin

```
+----------------+------------------------------------------------------------+
| Property       | Value                                                      |
+----------------+------------------------------------------------------------+
| Namespace      | media                                                      |
| Service        | phoenyxfin                                                 |
| Container Port | 8096                                                       |
| Service Type   | ClusterIP                                                  |
| External URL   | https://jellyfin.phoenyxlab.xyz                            |
| Dependencies   | none                                                       |
| Storage        | jellyfin-config PVC (10Gi), /media/movies, /media/music,   |
|                | /media/tv                                                  |
+----------------+------------------------------------------------------------+
```

## lidarr

```
+----------------+------------------------------------------------------------+
| Property       | Value                                                      |
+----------------+------------------------------------------------------------+
| Namespace      | media                                                      |
| Service        | lidarr                                                     |
| Container Port | 8686                                                       |
| Service Type   | ClusterIP                                                  |
| External URL   | https://lidarr.phoenyxlab.xyz                              |
| Dependencies   | qbittorrent (downloads), slskd (downloads)                 |
| Storage        | lidarr-config PVC (5Gi), /downloads/qbittorrent,           |
|                | /downloads/slskd/completed, /media/music                   |
+----------------+------------------------------------------------------------+
```

## musicseerr

```
+----------------+------------------------------------------------------------+
| Property       | Value                                                      |
+----------------+------------------------------------------------------------+
| Namespace      | media                                                      |
| Service        | musicseerr                                                 |
| Container Port | 8688                                                       |
| Service Type   | ClusterIP                                                  |
| External URL   | https://musicseerr.phoenyxlab.xyz                          |
| Dependencies   | music library                                              |
| Storage        | musicseerr PVC (1Gi), /media/music                         |
+----------------+------------------------------------------------------------+
```

## navidrome

```
+----------------+------------------------------------------------------------+
| Property       | Value                                                      |
+----------------+------------------------------------------------------------+
| Namespace      | media                                                      |
| Service        | navidrome                                                  |
| Container Port | 4533                                                       |
| Service Type   | ClusterIP                                                  |
| External URL   | https://navidrome.phoenyxlab.xyz                           |
| Dependencies   | music library                                              |
| Storage        | navidrome-config PVC (10Gi), /media/music                  |
+----------------+------------------------------------------------------------+
```

## pinepods

```
+----------------+------------------------------------------------------------+
| Property       | Value                                                      |
+----------------+------------------------------------------------------------+
| Namespace      | media                                                      |
| Service        | pinepods                                                   |
| Container Port | 8040                                                       |
| Service Type   | ClusterIP                                                  |
| External URL   | https://pinepods.phoenyxlab.xyz                            |
| Dependencies   | pinepods-postgres, pinepods-valkey                         |
| Storage        | pinepods-data PVC (20Gi)                                   |
+----------------+------------------------------------------------------------+
```

### sub-services

**pinepods-postgres**
```
+----------------+------------------------------------------------------------+
| Property       | Value                                                      |
+----------------+------------------------------------------------------------+
| Namespace      | media                                                      |
| Service        | pinepods-postgres                                          |
| Container Port | 5432                                                       |
| Service Type   | ClusterIP                                                  |
| Dependencies   | none                                                       |
| Storage        | pinepods-postgres PVC (5Gi)                                |
+----------------+------------------------------------------------------------+
```

**pinepods-valkey**
```
+----------------+------------------------------------------------------------+
| Property       | Value                                                      |
+----------------+------------------------------------------------------------+
| Namespace      | media                                                      |
| Service        | pinepods-valkey                                            |
| Container Port | 6379                                                       |
| Service Type   | ClusterIP                                                  |
| Dependencies   | none                                                       |
| Storage        | none                                                       |
+----------------+------------------------------------------------------------+
```

## radarr

```
+----------------+------------------------------------------------------------+
| Property       | Value                                                      |
+----------------+------------------------------------------------------------+
| Namespace      | media                                                      |
| Service        | radarr                                                     |
| Container Port | 7878                                                       |
| Service Type   | ClusterIP                                                  |
| External URL   | https://radarr.phoenyxlab.xyz                              |
| Dependencies   | qbittorrent (downloads)                                    |
| Storage        | radarr-config PVC (5Gi), /downloads/qbittorrent,           |
|                | /media/movies                                              |
+----------------+------------------------------------------------------------+
```

## seerr

```
+----------------+------------------------------------------------------------+
| Property       | Value                                                      |
+----------------+------------------------------------------------------------+
| Namespace      | media                                                      |
| Service        | seerr                                                      |
| Container Port | 5055                                                       |
| Service Type   | ClusterIP                                                  |
| External URL   | https://seerr.phoenyxlab.xyz                               |
| Dependencies   | radarr (API), sonarr (API)                                 |
| Storage        | seerr-config PVC (1Gi)                                     |
+----------------+------------------------------------------------------------+
```

## sonarr

```
+----------------+------------------------------------------------------------+
| Property       | Value                                                      |
+----------------+------------------------------------------------------------+
| Namespace      | media                                                      |
| Service        | sonarr                                                     |
| Container Port | 8989                                                       |
| Service Type   | ClusterIP                                                  |
| External URL   | https://sonarr.phoenyxlab.xyz                              |
| Dependencies   | qbittorrent (downloads)                                    |
| Storage        | sonarr-config PVC (5Gi), /downloads/qbittorrent,           |
|                | /media/tv                                                  |
+----------------+------------------------------------------------------------+
```

## soulsync

```
+----------------+------------------------------------------------------------+
| Property       | Value                                                      |
+----------------+------------------------------------------------------------+
| Namespace      | media                                                      |
| Service        | soulsync                                                   |
| Container Port | 8008                                                       |
| Service Type   | ClusterIP                                                  |
| External URL   | https://soulsync.phoenyxlab.xyz                            |
| Dependencies   | slskd (downloads), music library                           |
| Storage        | soulsync-config PVC (2Gi), /downloads/slskd, /media/music  |
+----------------+------------------------------------------------------------+
```

---

# infra

## homarr

```
+----------------+------------------------------------------------------------+
| Property       | Value                                                      |
+----------------+------------------------------------------------------------+
| Namespace      | infra                                                      |
| Service        | homarr                                                     |
| Container Port | 7575                                                       |
| Service Type   | ClusterIP                                                  |
| External URL   | https://homarr.phoenyxlab.xyz                              |
| Dependencies   | none                                                       |
| Storage        | homarr-config PVC (1Gi)                                    |
+----------------+------------------------------------------------------------+
```

## homepage

```
+----------------+------------------------------------------------------------+
| Property       | Value                                                      |
+----------------+------------------------------------------------------------+
| Namespace      | infra                                                      |
| Service        | homepage                                                   |
| Container Port | 3000                                                       |
| Service Type   | ClusterIP                                                  |
| External URL   | https://homepage.phoenyxlab.xyz                            |
| Dependencies   | all services (widget integrations via API)                 |
| Storage        | homepage-assets PVC (1Gi)                                  |
+----------------+------------------------------------------------------------+
```

RBAC: ServiceAccount `homepage`, ClusterRole `homepage` (get/list: namespaces, pods, nodes, ingresses, ingressroutes, httproutes, gateways, metrics), ClusterRoleBinding `homepage`.

## notifiarr

```
+----------------+------------------------------------------------------------+
| Property       | Value                                                      |
+----------------+------------------------------------------------------------+
| Namespace      | infra                                                      |
| Service        | notifiarr                                                  |
| Container Port | 5454                                                       |
| Service Type   | ClusterIP                                                  |
| External URL   | https://notifiarr.phoenyxlab.xyz                           |
| Dependencies   | *arr apps, qbittorrent (integrations)                      |
| Storage        | notifiarr-config PVC (1Gi)                                 |
+----------------+------------------------------------------------------------+
```

## pihole

```
+----------------+------------------------------------------------------------+
| Property       | Value                                                      |
+----------------+------------------------------------------------------------+
| Namespace      | infra                                                      |
| Service        | pihole (web), pihole-dns (DNS)                             |
| Container Port | 80 (web), 53 TCP/UDP (DNS)                                 |
| Service Type   | ClusterIP (web), LoadBalancer (DNS)                        |
| External URL   | https://pihole.phoenyxlab.xyz                              |
| Dependencies   | none                                                       |
| Storage        | pihole-config PVC (5Gi)                                    |
+----------------+------------------------------------------------------------+
```

### middleware

- `pihole-admin-redirect`: Traefik Middleware (redirectRegex) — redirects `https://pihole.phoenyxlab.xyz/` to `https://pihole.phoenyxlab.xyz/admin/`

## vaultwarden

```
+----------------+------------------------------------------------------------+
| Property       | Value                                                      |
+----------------+------------------------------------------------------------+
| Namespace      | infra                                                      |
| Service        | vaultwarden                                                |
| Container Port | 80                                                         |
| Service Type   | ClusterIP                                                  |
| External URL   | https://vaultwarden.phoenyxlab.xyz                         |
| Dependencies   | none                                                       |
| Storage        | vaultwarden-config PVC (1Gi)                               |
+----------------+------------------------------------------------------------+
```

Signups disabled, admin panel enabled.

---

# monitoring

## glances

```
+----------------+------------------------------------------------------------+
| Property       | Value                                                      |
+----------------+------------------------------------------------------------+
| Namespace      | monitoring                                                 |
| Service        | glances                                                    |
| Container Port | 61208                                                      |
| Service Type   | ClusterIP                                                  |
| External URL   | https://glances.phoenyxlab.xyz                             |
| Dependencies   | none                                                       |
| Storage        | glances-config PVC (1Gi)                                   |
+----------------+------------------------------------------------------------+
```

Host PID namespace enabled. SMART plugin active.

## uptime kuma

```
+----------------+------------------------------------------------------------+
| Property       | Value                                                      |
+----------------+------------------------------------------------------------+
| External URL   | https://uptime.phoenyxlab.xyz                              |
| Internal URL   | https://uptime-internal.phoenyxlab.xyz (Tailscale)         |
| Hosting        | OCI VM (external to cluster, connected via Tailscale)      |
| Usage          | external uptime monitoring for all services                |
+----------------+------------------------------------------------------------+
```

Not deployed in-cluster. The OCI VM is part of the Tailscale network; the homepage widget accesses it via the internal Tailscale URL.

---

# infrastructure

## traefik

```
+----------------+------------------------------------------------------------+
| Property       | Value                                                      |
+----------------+------------------------------------------------------------+
| Namespace      | kube-system                                                |
| Service        | traefik (api@internal)                                     |
| External URL   | https://traefik.phoenyxlab.xyz                             |
| Dependencies   | cert-manager (TLS)                                         |
| Storage        | none                                                       |
+----------------+------------------------------------------------------------+
```

Primary ingress controller. All ingresses use `traefik` ingressClass.

### TLS

- `TLSStore` (`default`) in `kube-system` — default certificate: `phoenyxlab-wildcard`

### middleware

Standard entrypoint: `websecure`. All routes use TLS termination at the edge.

## cert-manager

```
+----------------+------------------------------------------------------------+
| Property       | Value                                                      |
+----------------+------------------------------------------------------------+
| Namespace      | cert-manager                                               |
| Type           | HelmRelease (jetstack/cert-manager v1.20.3)                |
| Dependencies   | godaddy-webhook (DNS-01 solver)                            |
| Storage        | none                                                       |
+----------------+------------------------------------------------------------+
```

### resources

- `ClusterIssuer`: `letsencrypt-prod` — ACME via GoDaddy DNS-01, email: wolfinthedepths@proton.me
- `Certificate`: `phoenyxlab-wildcard` in `kube-system` — wildcard `*.phoenyxlab.xyz` + root domain, stored as secret `phoenyxlab-wildcard`, renew before 240h

## godaddy-webhook

```
+----------------+------------------------------------------------------------+
| Property       | Value                                                      |
+----------------+------------------------------------------------------------+
| Namespace      | cert-manager                                               |
| Type           | HelmRelease (snowdrop/godaddy-webhook)                     |
| GroupName      | acme.phoenyxlab.xyz                                        |
| Dependencies   | GoDaddy API key (SOPS-encrypted secret)                    |
| Storage        | none                                                       |
+----------------+------------------------------------------------------------+
```

DNS-01 ACME solver for cert-manager.

## backup

```
+----------------+------------------------------------------------------------+
| Property       | Value                                                      |
+----------------+------------------------------------------------------------+
| Namespace      | backup                                                     |
| Type           | CronJob (restic)                                           |
| Schedule       | 0 2 * * * (Asia/Kolkata)                                   |
| Repository     | s3://...r2.cloudflarestorage.com/phoenyxlab-backups        |
| Retention      | keep-weekly 8, keep-monthly 12, prune                      |
| Dependencies   | Cloudflare R2 (S3-compatible)                              |
| Storage        | /var/lib/rancher/k3s/storage (hostPath)                    |
+----------------+------------------------------------------------------------+
```

Restic backups to Cloudflare R2. Encrypted with SOPS-stored credentials.

## kube-prometheus-stack

```
+----------------+------------------------------------------------------------+
| Property       | Value                                                      |
+----------------+------------------------------------------------------------+
| Namespace      | monitoring                                                 |
| Type           | HelmRelease (prometheus-community/kube-prometheus-stack    |
|                |   v87.4.0)                                                 |
| Dependencies   | all services (metrics endpoints)                           |
| Storage        | grafana PVC (10Gi)                                         |
+----------------+------------------------------------------------------------+
```

### components

```
+-----------------------+-------------+-------------------------------------|
| Component             | Enabled     | Ingress URL                         |
+-----------------------+-------------+-------------------------------------|
| Alertmanager          | Yes         | https://alertmanager.phoenyxlab.xyz |
| Grafana               | Yes         | https://grafana.phoenyxlab.xyz      |
| Prometheus            | Yes         | https://prometheus.phoenyxlab.xyz   |
| Node Exporter         | Yes         | —                                   |
| kube-state-metrics    | Yes         | —                                   |
| kubeProxy             | No          | —                                   |
| kubeControllerManager | No          | —                                   |
| kubeScheduler         | No          | —                                   |
+-----------------------+-------------+-------------------------------------|
```

Alertmanager uses Discord webhook receiver. Grafana admin user: wolf. Prometheus scrapes traefik via ServiceMonitor in `monitoring`.

---

# dependency graph

```
gluetun (VPN)
  └─► prowlarr (initContainer)
  └─► qbittorrent (initContainer + port forwarding)

qbittorrent (downloader)
  └─► radarr (downloads mount)
  └─► sonarr (downloads mount)
  └─► lidarr (downloads mount)

slskd (Soulseek)
  └─► lidarr (downloads mount)
  └─► soulsync (downloads mount)

music library
  └─► lidarr (music mount)
  └─► navidrome (music mount)
  └─► jellyfin (music mount)
  └─► musicseerr (music mount)

freshrss-postgres
  └─► freshrss

pinepods-postgres + pinepods-valkey
  └─► pinepods

cert-manager + godaddy-webhook
  └─► all ingress TLS (wildcard *.phoenyxlab.xyz)

traefik
  └─► all ingress routing

homepage
  └─► all services (widget integrations via API)

kube-prometheus-stack
  └─► all services (metrics/observability)
```
