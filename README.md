# Cloudflared Helm Chart

A minimal, security-hardened Helm chart for deploying [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/) (`cloudflared`) on Kubernetes.

## Features

- Security-first defaults: non-root, read-only filesystem, dropped capabilities, seccomp profile
- PodDisruptionBudget for high availability
- Zero-downtime rolling updates
- Liveness and readiness probes via the built-in metrics server
- Pod anti-affinity to spread replicas across nodes
- Support for existing secrets (Vault, External Secrets, SealedSecret, etc.)
- Optional NetworkPolicy (egress-only)
- Optional Prometheus metrics Service and ServiceMonitor
- Optional OpenTelemetry annotations for automatic scraping

## Prerequisites

- Kubernetes 1.23+
- Helm 3.x
- A [Cloudflare Tunnel token](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/get-started/create-remote-tunnel/)

## Installation

### With tunnel token

```bash
helm install cloudflared ./cloudflared \
  --namespace cloudflared \
  --create-namespace \
  --set cloudflared.tunnelToken="<your-tunnel-token>"
```

### With an existing secret

```bash
kubectl create secret generic my-tunnel-secret \
  --from-literal=tunnelToken="<your-tunnel-token>" \
  -n cloudflared

helm install cloudflared ./cloudflared \
  --namespace cloudflared \
  --create-namespace \
  --set cloudflared.existingSecret=my-tunnel-secret
```

### With OpenTelemetry and metrics

```bash
helm install cloudflared ./cloudflared \
  --namespace cloudflared \
  --create-namespace \
  --set cloudflared.tunnelToken="<your-tunnel-token>" \
  --set opentelemetry.enabled=true \
  --set metrics.service.enabled=true \
  --set metrics.serviceMonitor.enabled=true
```

## Uninstall

```bash
helm uninstall cloudflared -n cloudflared
```

## Parameters

### General

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of cloudflared replicas | `2` |
| `image.repository` | Container image repository | `cloudflare/cloudflared` |
| `image.tag` | Container image tag | Chart `appVersion` |
| `image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `imagePullSecrets` | Image pull secrets | `[]` |
| `nameOverride` | Override chart name | `""` |
| `fullnameOverride` | Override full release name | `""` |

### Cloudflared

| Parameter | Description | Default |
|-----------|-------------|---------|
| `cloudflared.tunnelToken` | Cloudflare Tunnel token (required if `existingSecret` is not set) | `""` |
| `cloudflared.existingSecret` | Name of an existing Secret containing a `tunnelToken` key | `""` |

### Service Account

| Parameter | Description | Default |
|-----------|-------------|---------|
| `serviceAccount.enabled` | Create a service account | `true` |
| `serviceAccount.name` | Service account name | `cloudflared-sa` |
| `serviceAccount.annotations` | Service account annotations | `{}` |
| `serviceAccount.automountServiceAccountToken` | Automount API token | `false` |

### Security

| Parameter | Description | Default |
|-----------|-------------|---------|
| `podSecurityContext.runAsNonRoot` | Run as non-root user | `true` |
| `podSecurityContext.runAsUser` | User ID | `65532` |
| `podSecurityContext.runAsGroup` | Group ID | `65532` |
| `securityContext.allowPrivilegeEscalation` | Allow privilege escalation | `false` |
| `securityContext.readOnlyRootFilesystem` | Read-only root filesystem | `true` |
| `securityContext.capabilities.drop` | Dropped capabilities | `["ALL"]` |
| `securityContext.seccompProfile.type` | Seccomp profile | `RuntimeDefault` |

### High Availability

| Parameter | Description | Default |
|-----------|-------------|---------|
| `strategy.type` | Deployment strategy | `RollingUpdate` |
| `strategy.rollingUpdate.maxUnavailable` | Max unavailable pods during update | `0` |
| `strategy.rollingUpdate.maxSurge` | Max extra pods during update | `1` |
| `podDisruptionBudget.enabled` | Enable PDB | `true` |
| `podDisruptionBudget.minAvailable` | Minimum available pods | `1` |
| `terminationGracePeriodSeconds` | Grace period for pod termination | `30` |
| `affinity` | Affinity rules (anti-affinity preferred by default) | see `values.yaml` |

### Probes

| Parameter | Description | Default |
|-----------|-------------|---------|
| `livenessProbe.httpGet.path` | Liveness probe path | `/ready` |
| `livenessProbe.httpGet.port` | Liveness probe port | `2000` |
| `readinessProbe.httpGet.path` | Readiness probe path | `/ready` |
| `readinessProbe.httpGet.port` | Readiness probe port | `2000` |

### Resources

| Parameter | Description | Default |
|-----------|-------------|---------|
| `resources.requests.cpu` | CPU request | `50m` |
| `resources.requests.memory` | Memory request | `64Mi` |
| `resources.limits.cpu` | CPU limit | `800m` |
| `resources.limits.memory` | Memory limit | `256Mi` |

### Network Policy

| Parameter | Description | Default |
|-----------|-------------|---------|
| `networkPolicy.enabled` | Enable NetworkPolicy (egress: QUIC/7844, HTTPS/443, DNS/53) | `false` |

### Metrics and Observability

| Parameter | Description | Default |
|-----------|-------------|---------|
| `metrics.service.enabled` | Create a ClusterIP Service for metrics | `false` |
| `metrics.service.port` | Metrics service port | `2000` |
| `metrics.serviceMonitor.enabled` | Create a Prometheus ServiceMonitor | `false` |
| `metrics.serviceMonitor.interval` | Scrape interval | `30s` |
| `metrics.serviceMonitor.additionalLabels` | Additional labels on the ServiceMonitor | `{}` |
| `opentelemetry.enabled` | Add prometheus.io annotations for OTEL collector discovery | `false` |

### Scheduling

| Parameter | Description | Default |
|-----------|-------------|---------|
| `nodeSelector` | Node selector | `{}` |
| `tolerations` | Tolerations | `[]` |
| `podAnnotations` | Pod annotations | `{}` |
| `podLabels` | Pod labels | `{}` |

## License

[MIT](LICENSE)
