# Cloudflared Helm Chart

[![Artifact Hub](https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/cloudflared)](https://artifacthub.io/packages/search?repo=cloudflared)

A minimal, security-hardened Helm chart for deploying [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/) (`cloudflared`) on Kubernetes.

## Features

- Security-first defaults: non-root, read-only filesystem, writable `/tmp`, dropped capabilities, seccomp profile
- PodDisruptionBudget for high availability
- Zero-downtime rolling updates
- Decoupled probes: liveness (tcpSocket) tolerates edge outages, readiness (httpGet /ready) gates traffic
- Pod anti-affinity to spread replicas across nodes
- Support for existing secrets (Vault, External Secrets, SealedSecret, etc.)
- ICMP proxying (ping, traceroute) to private networks through a safe sysctl
- Any cloudflared flag through `extraEnv`
- Optional NetworkPolicy with configurable namespace-level egress
- Optional Prometheus metrics Service and ServiceMonitor
- Optional OpenTelemetry annotations for automatic scraping

## Prerequisites

- Kubernetes 1.23+
- Helm 3.x
- A [Cloudflare Tunnel token](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/get-started/create-remote-tunnel/)

## Installation

Add the Helm repository:

```bash
helm repo add cloudflared https://theobs-digital.github.io/cloudflared-helm-chart
helm repo update
```

To install from a clone of this repository instead, replace `cloudflared/cloudflared` with `./cloudflared` in the commands below.

### With tunnel token

```bash
helm install cloudflared cloudflared/cloudflared \
  --namespace cloudflared \
  --create-namespace \
  --set cloudflared.tunnelToken="<your-tunnel-token>"
```

### With an existing secret

```bash
kubectl create secret generic my-tunnel-secret \
  --from-literal=tunnelToken="<your-tunnel-token>" \
  -n cloudflared

helm install cloudflared cloudflared/cloudflared \
  --namespace cloudflared \
  --create-namespace \
  --set cloudflared.existingSecret=my-tunnel-secret
```

### With OpenTelemetry and metrics

```bash
helm install cloudflared cloudflared/cloudflared \
  --namespace cloudflared \
  --create-namespace \
  --set cloudflared.tunnelToken="<your-tunnel-token>" \
  --set opentelemetry.enabled=true \
  --set metrics.service.enabled=true \
  --set metrics.serviceMonitor.enabled=true
```

### With extra cloudflared flags

Every [`tunnel run` parameter](https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/configure-tunnels/run-parameters/) has a `TUNNEL_*` environment variable. Pass them through `cloudflared.extraEnv`:

```yaml
cloudflared:
  extraEnv:
    - name: TUNNEL_LOG_OUTPUT          # JSON logs, for log collectors
      value: json
    - name: TUNNEL_EDGE_IP_VERSION     # IPv4-only clusters (default is auto since cloudflared 2026.4.0)
      value: "4"
    - name: TUNNEL_TRANSPORT_PROTOCOL  # when UDP/7844 is blocked
      value: http2
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
| `cloudflared.extraEnv` | Extra environment variables, e.g. `TUNNEL_*` equivalents of `tunnel run` flags | `[]` |

### Service Account

| Parameter | Description | Default |
|-----------|-------------|---------|
| `serviceAccount.enabled` | Create a service account | `true` |
| `serviceAccount.name` | Service account name (empty = derived from release fullname) | `""` |
| `serviceAccount.annotations` | Service account annotations | `{}` |
| `serviceAccount.automountServiceAccountToken` | Automount API token | `false` |

### Security

| Parameter | Description | Default |
|-----------|-------------|---------|
| `podSecurityContext.runAsNonRoot` | Run as non-root user | `true` |
| `podSecurityContext.runAsUser` | User ID | `65532` |
| `podSecurityContext.runAsGroup` | Group ID | `65532` |
| `podSecurityContext.sysctls` | Pod sysctls; the default enables ICMP proxying (set `[]` to disable) | `net.ipv4.ping_group_range: "65532 65532"` |
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
| `podDisruptionBudget.enabled` | Enable PDB (auto-disabled when `replicaCount` is 1) | `true` |
| `podDisruptionBudget.minAvailable` | Minimum available pods | `1` |
| `terminationGracePeriodSeconds` | Grace period for pod termination (must exceed cloudflared's `--grace-period`) | `45` |
| `affinity` | Affinity rules (anti-affinity preferred by default) | see `values.yaml` |

### Probes

| Parameter | Description | Default |
|-----------|-------------|---------|
| `livenessProbe.tcpSocket.port` | Liveness probe port (tcpSocket) | `2000` |
| `livenessProbe.failureThreshold` | Failures before restart (~2 min tolerance) | `6` |
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
| `networkPolicy.enabled` | Enable NetworkPolicy | `false` |
| `networkPolicy.allowedNamespaces` | Namespaces cloudflared can reach (`["*"]` for all) | `["*"]` |
| `networkPolicy.additionalEgress` | Additional raw egress rules for fine-grained control (pod labels, IP blocks, ports) | `[]` |

When enabled, the NetworkPolicy allows:
- **Ingress**: TCP/2000 (metrics and probes) from any source
- **Egress**: UDP/7844 + TCP/7844 (Cloudflare tunnel), TCP/443 (Cloudflare API), UDP+TCP/53 (DNS)
- **Egress backends**: all pods in `allowedNamespaces` on all ports

Set `allowedNamespaces` to specific namespaces (e.g. `["production", "staging"]`) to restrict which backends cloudflared can reach. For fine-grained control (pod labels, IP blocks, specific ports), use `additionalEgress` with raw NetworkPolicy rules.

### Metrics and Observability

| Parameter | Description | Default |
|-----------|-------------|---------|
| `metrics.service.enabled` | Create a ClusterIP Service for metrics | `false` |
| `metrics.service.port` | Metrics service port | `2000` |
| `metrics.serviceMonitor.enabled` | Create a Prometheus ServiceMonitor (requires `metrics.service.enabled`) | `false` |
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

## Design notes

- **Liveness vs readiness**: liveness uses `tcpSocket` (process alive?) while readiness uses `httpGet /ready` (tunnel connected?). This prevents crashloops during Cloudflare edge outages — pods stay alive but are removed from Services until the tunnel reconnects.
- **Anti-affinity**: `preferredDuringScheduling` by default so the chart works on single-node clusters. On multi-node clusters, replicas are spread across nodes but scheduling is not blocked if spreading is impossible.
- **PDB**: auto-disabled when `replicaCount: 1` to avoid blocking node drains.
- **Security context**: `runAsNonRoot`/`runAsUser`/`runAsGroup` are set at both pod and container level intentionally for defense-in-depth.
- **ICMP sysctl**: `net.ipv4.ping_group_range` lets the non-root process open ICMP sockets, so ping and traceroute reach private networks behind the tunnel (same setting as Cloudflare's Kubernetes guide). It is a safe sysctl, allowed by the kubelet and Pod Security `restricted` by default. The range must include `runAsGroup`. On clusters that forbid pod sysctls (admission policy, sandboxed runtime), set `podSecurityContext.sysctls: []`.
- **Termination grace period**: on SIGTERM, cloudflared stops accepting requests and drains in-flight ones for up to `--grace-period` (30s by default). `terminationGracePeriodSeconds` is set above that so the kubelet's SIGKILL does not cut the drain. If you raise `TUNNEL_GRACE_PERIOD`, raise `terminationGracePeriodSeconds` too.

## License

[MIT](LICENSE)
