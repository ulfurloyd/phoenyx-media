
<h1 align="center">phoenyxlab</h1>

<h2 align="center">
    <img src="./.github/ulfurloyd.jpg" width="240" alt="ulfurloyd" /> <br />
</h2>

[![.github/workflows/validate-images.yaml](https://github.com/ulfurloyd/phoenyxlab/actions/workflows/validate-images.yaml/badge.svg?branch=main)](https://github.com/ulfurloyd/phoenyxlab/actions/workflows/validate-images.yaml) [![Validate Kubernetes](https://github.com/ulfurloyd/phoenyxlab/actions/workflows/validate-kubernetes.yaml/badge.svg)](https://github.com/ulfurloyd/phoenyxlab/actions/workflows/validate-kubernetes.yaml)

personal GitOps-managed homelab running on Kubernetes


## overview

phoenyxlab is my self-hosted platform for media, infrastructure,
monitoring, automation, and experimentation. What began as a way to
learn Kubernetes has evolved into the primary operating environment
for my home network.

the cluster is managed declaratively using FluxCD and GitOps
principles: Git is the single source of truth and Flux continuously
reconciles desired state.

## philosophy

the primary goals of this project are:

- learn Kubernetes by operating real workloads
- maintain a fully declarative infrastructure
- minimize manual cluster changes
- ensure services are reproducible and recoverable
- keep application state separate from cluster state
- treat infrastructure as software

in practice this means:

- Kubernetes manifests live in Git
- FluxCD continuously reconciles cluster state
- secrets are encrypted using SOPS + age
- application data lives on PVCs
- backups are performed automatically
- upgrades are proposed automatically through Renovate

## architecture

```text
Git
 │
 ▼
FluxCD
 │
 ├── infrastructure
 │   ├── traefik
 │   ├── cert-manager
 │   ├── monitoring
 │   └── backup
 │
 └── applications
     ├── infra
     ├── media
     ├── downloads
     └── monitoring
```

## repo structure

```
.
├── apps/
│   ├── downloads/
│   ├── infra/
│   ├── media/
│   └── monitoring/
│
├── infrastructure/
│   ├── backup/
│   ├── cert-manager/
│   ├── monitoring/
│   ├── network-policies/
│   └── traefik/
│
├── clusters/
│   └── phoenyxlab/
│
└── docs/
```

### applications

application manifests follow a resource-oriented layout:

```
application/
├── deployment.yaml
├── service.yaml
├── ingress.yaml
├── pvc.yaml
├── secret.yaml
└── kustomization.yaml
```

applications requiring supporting services keep those resources
alongside the application:

```
pinepods/
├── postgres-deployment.yaml
├── postgres-service.yaml
├── valkey-deployment.yaml
├── deployment.yaml
└── ...
```

## cluster infrastructure

### networking

- traefik ingress controller
- wildcard TLS certificate via cert-manager
- Pi-hole for internal DNS
- private access via Tailscale
- NetworkPolicies: default-deny ingress, explicit allow rules by namespace

### GitOps

- FluxCD reconciliation
- Flux alerts to Discord
- SOPS + age encrypted secrets
- Renovate automated dependency updates

### monitoring

- kube-prometheus-stack
- prometheus
- grafana
- alertmanager
- node-exporter
- glances
- speedtest-tracker
- uptime kuma (external)

### backup

- restic cronjobs
- encrypted backups
- cloudflare R2 storage
- monthly retention policies

### namespaces

```
+---------------+-----------------------------------+
| namespace     | purpose                           |
+---------------+-----------------------------------+
| infra         | core self-hosted infrastructure   |
| media         | media management and consumption  |
| downloads     | acquisition pipeline              |
| monitoring    | cluster and host observability    |
| cert-manager  | TLS certificate management        |
| backup        | backup infrastructure             |
+---------------+-----------------------------------+
```

### service inventory

#### self-hosted applications

```
+--------------------+-------------+---------------------+
| hostname           | namespace   | description         |
|--------------------+-------------+---------------------|
| homarr             | infra       | dashboard           |
| homepage           | infra       | dashboard           |
| notifiarr          | infra       | notification relay  |
| pi-hole            | infra       | DNS and adblock     |
| vaultwarden        | infra       | password manager    |
| freshrss           | media       | RSS Reader          |
| jellyfin           | media       | media server        |
| lidarr             | media       | music acquisition   |
| navidrome          | media       | music server        |
| pinepods           | media       | podcast server      |
| radarr             | media       | movie management    |
| seerr              | media       | media requests      |
| droppedneedle      | media       | music requests      |
| sonarr             | media       | tv management       |
| soulsync           | media       | music syncing       |
| jackett            | downloads   | indexer management  |
| prowlarr           | downloads   | indexer management  |
| qbittorrent        | downloads   | torrent client      |
| slskd              | downloads   | soulseek client     |
| glances            | monitoring  | system stats        |
| speedtest-tracker  | monitoring  | network speed tests |
+--------------------+-------------+---------------------+
```

#### cluster infrastructure

```
+---------------------+-----------------------------+
| component           | purpose                     |
+---------------------+-----------------------------+
| traefik             | ingress controller          |
| FluxCD              | GitOps                      |
| cert-manager        | TLS certificate management  |
| prometheus          | metrics collection          |
| grafana             | dashboards                  |
| alertmanager        | alert routing               |
| node-exporter       | host metrics                |
| kube-state-metrics  | kubernetes metrics          |
| restic              | backups                     |
| SOPS + age          | secret management           |
| renovate            | dependency updates          |
+---------------------+-----------------------------+
```

#### external services

```
+----------------------+-----------------------------+
| service              | purpose                     |
+----------------------+-----------------------------+
| uptime kuma (OCI VM) | external uptime monitoring  |
| cloudflare R2        | backup storage              |
| GoDaddy DNS          | authoritative DNS           |
| Tailscale            | private remote access       |
+----------------------+-----------------------------+
```

### media automation

```
Request
   │
   ▼
Seerr / Droppedneedle
   │
   ▼
Sonarr / Radarr / Lidarr
   │
   ▼
Prowlarr
   │
   ▼
qBittorrent / slskd
   │
   ▼
Import → Organize → Cleanup
   │
   ▼
Jellyfin / Navidrome
```

### validation and automation

- kubeconform validates manifests
- container image tags are verified in CI
- Renovate tracks images and Helm charts
- Flux continuously reconciles desired state
- deployment failures generate Discord alerts

### backup and recovery

the cluster is designed around recoverability.

- declarative state exists in Git
- secrets exist as SOPS-encrypted manifests
- application data exists on PVCs
- persistent data is backed up using Restic
- storage uses `Retain` reclaim policies where appropriate

cluster recovery consists of:

1. install k3s
2. install FluxCD
3. restore the SOPS age key
4. bootstrap Flux
5. allow Flux to reconcile infrastructure and applications
6. restore PVC data if necessary
