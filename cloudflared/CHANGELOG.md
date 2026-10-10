# Changelog

## [0.6.0] - 2026-10-10

### Added
- `cloudflared.extraEnv` to pass any `tunnel run` flag through its `TUNNEL_*` environment variable (e.g. `TUNNEL_LOG_OUTPUT=json`, `TUNNEL_EDGE_IP_VERSION=4`)
- `net.ipv4.ping_group_range` sysctl in the default `podSecurityContext`, as in Cloudflare's Kubernetes guide: ping and traceroute now reach private networks behind the tunnel
- README: installation from the Helm repository (`helm repo add`)

### Changed
- Bump cloudflared to 2026.10.0 (CVE patches)
- `terminationGracePeriodSeconds` default raised from 30 to 45, so the kubelet no longer kills cloudflared before its own 30s grace period ends
- **Note**: clusters that forbid pod sysctls (admission policy, sandboxed runtime) must set `podSecurityContext.sysctls: []`, otherwise new pods are rejected and the rollout stays on the old ones

## [0.5.1] - 2026-06-20

### Changed
- Bump cloudflared to 2026.6.1 (CVE patches)

## [0.5.0] - 2026-06-14

### Fixed
- ServiceAccount name now derives from the release fullname (default `name: ""`) instead of a hardcoded `cloudflared-sa`, which collided when several releases shared a namespace
- Metrics Service `targetPort` now points at the named container port `metrics` instead of following `metrics.service.port`, so changing the service port no longer breaks routing

### Changed
- **Breaking**: existing installs get a renamed ServiceAccount on upgrade (`cloudflared-sa` → `<release>-cloudflared`); re-apply any annotations (e.g. IRSA/Workload Identity) set on the old SA

## [0.4.5] - 2026-06-14

### Changed
- Bump cloudflared to 2026.6.0 (CVE patches)

## [0.4.4] - 2026-06-04

### Fixed
- Include README.md and CHANGELOG.md in chart package for ArtifactHub display

## [0.4.3] - 2026-06-03

### Fixed
- Chart icon URL updated (previous favicon-128.png returned 404 on ArtifactHub)

## [0.4.2] - 2026-06-03

### Fixed
- First chart-releaser publish (version bump to trigger GitHub Pages index.yaml)

## [0.4.1] - 2026-06-03

### Fixed
- Liveness probe decoupled from readiness: uses `tcpSocket` instead of `httpGet /ready` to avoid crashloop during Cloudflare edge outages
- Added TCP/7844 egress in NetworkPolicy for HTTP/2 fallback
- ServiceMonitor now fails at install if `metrics.service.enabled` is not set

## [0.4.0] - 2026-06-03

### Added
- Writable `/tmp` volume (emptyDir) for readOnlyRootFilesystem compatibility
- `networkPolicy.allowedNamespaces` to control egress to cluster backends (default: `["*"]` — all namespaces)
- `securityContext.runAsUser` and `securityContext.runAsGroup` in default values

### Fixed
- NetworkPolicy `from: []` and `to: []` rules were silently blocking all traffic instead of allowing it
- PodDisruptionBudget now auto-disables when `replicaCount` is 1 to avoid blocking node drains
- Tunnel token validation uses `required` instead of a default placeholder that would fail at runtime

## [0.3.0] - 2026-05-31

### Added
- `existingSecret` support to reference an external Secret (Vault, External Secrets, SealedSecret, etc.)
- Pod anti-affinity `preferredDuringScheduling` to spread replicas across nodes
- Configurable `terminationGracePeriodSeconds` (default 30s)
- Optional egress-only NetworkPolicy (QUIC/7844, HTTPS/443, DNS/53)
- Optional metrics Service (ClusterIP on port 2000)
- Optional ServiceMonitor with `additionalLabels` for Prometheus Operator compatibility
- Conditional OpenTelemetry support (prometheus.io annotations on pods)
- `containerPort` metrics declared when OpenTelemetry or metrics service is enabled

### Changed
- CPU limit increased from 200m to 800m
- Memory limit increased from 128Mi to 256Mi
- NOTES.txt enriched with existingSecret, metrics and NetworkPolicy info
- Token error message clarified when `existingSecret` is not set

## [0.2.0]

### Added
- Deployment with liveness/readiness probes
- Secret for tunnel token
- ServiceAccount with automountServiceAccountToken: false
- PodDisruptionBudget
- Full security context (runAsNonRoot, readOnlyRootFilesystem, drop ALL, seccomp)
- Zero downtime rolling update
- Checksum/secret annotation for automatic restart

---

# Journal des modifications

## [0.6.0] - 2026-10-10

### Ajouts
- `cloudflared.extraEnv` pour passer n'importe quel flag de `tunnel run` via sa variable d'environnement `TUNNEL_*` (ex. `TUNNEL_LOG_OUTPUT=json`, `TUNNEL_EDGE_IP_VERSION=4`)
- Sysctl `net.ipv4.ping_group_range` dans le `podSecurityContext` par defaut, comme dans le guide Kubernetes de Cloudflare : ping et traceroute atteignent desormais les reseaux prives derriere le tunnel
- README : installation depuis le depot Helm (`helm repo add`)

### Modifications
- Mise a jour de cloudflared vers 2026.10.0 (correctifs CVE)
- `terminationGracePeriodSeconds` passe de 30 a 45 par defaut, pour que le kubelet ne tue plus cloudflared avant la fin de son propre delai de grace de 30s
- **Note** : les clusters qui interdisent les sysctls de pod (politique d'admission, runtime sandboxe) doivent definir `podSecurityContext.sysctls: []`, sinon les nouveaux pods sont refuses et le rollout reste sur les anciens

## [0.5.1] - 2026-06-20

### Modifications
- Mise a jour de cloudflared vers 2026.6.1 (correctifs CVE)

## [0.5.0] - 2026-06-14

### Corrections
- Le nom du ServiceAccount derive desormais du fullname de la release (defaut `name: ""`) au lieu d'un `cloudflared-sa` code en dur, qui entrait en collision lorsque plusieurs releases partageaient un namespace
- Le `targetPort` du Service metrics pointe maintenant vers le port conteneur nomme `metrics` au lieu de suivre `metrics.service.port`, donc changer le port du service ne casse plus le routage

### Modifications
- **Rupture** : les installs existants voient leur ServiceAccount renomme a l'upgrade (`cloudflared-sa` → `<release>-cloudflared`) ; reappliquer les annotations (ex. IRSA/Workload Identity) posees sur l'ancien SA

## [0.4.5] - 2026-06-14

### Modifications
- Mise a jour de cloudflared vers 2026.6.0 (correctifs CVE)

## [0.4.4] - 2026-06-04

### Corrections
- Inclusion de README.md et CHANGELOG.md dans le package du chart pour l'affichage sur ArtifactHub

## [0.4.3] - 2026-06-03

### Corrections
- URL de l'icone du chart corrigee (favicon-128.png retournait 404 sur ArtifactHub)

## [0.4.2] - 2026-06-03

### Corrections
- Premiere publication chart-releaser (bump de version pour generer index.yaml sur GitHub Pages)

## [0.4.1] - 2026-06-03

### Corrections
- Sonde liveness decouplée de readiness : utilise `tcpSocket` au lieu de `httpGet /ready` pour eviter les crashloops lors d'incidents edge Cloudflare
- Ajout TCP/7844 en egress dans la NetworkPolicy pour le fallback HTTP/2
- ServiceMonitor echoue a l'installation si `metrics.service.enabled` n'est pas active

## [0.4.0] - 2026-06-03

### Ajouts
- Volume `/tmp` writable (emptyDir) pour compatibilite readOnlyRootFilesystem
- `networkPolicy.allowedNamespaces` pour controler l'egress vers les backends du cluster (defaut : `["*"]` — tous les namespaces)
- `securityContext.runAsUser` et `securityContext.runAsGroup` dans les valeurs par defaut

### Corrections
- NetworkPolicy `from: []` et `to: []` bloquaient silencieusement tout le trafic au lieu de l'autoriser
- PodDisruptionBudget se desactive automatiquement quand `replicaCount` est 1 pour eviter de bloquer les drains de noeuds
- Validation du tunnel token avec `required` au lieu d'un placeholder qui echouait au runtime

## [0.3.0] - 2026-05-31

### Ajouts
- Support `existingSecret` pour referencer un Secret externe (Vault, External Secrets, SealedSecret, etc.)
- Anti-affinite pod `preferredDuringScheduling` pour repartir les replicas sur des nodes differents
- `terminationGracePeriodSeconds` configurable (defaut 30s)
- NetworkPolicy egress-only optionnelle (QUIC/7844, HTTPS/443, DNS/53)
- Service metrics optionnel (ClusterIP sur port 2000)
- ServiceMonitor optionnel avec `additionalLabels` pour compatibilite Prometheus Operator
- Support OpenTelemetry conditionnel (annotations prometheus.io sur les pods)
- `containerPort` metrics declare quand OpenTelemetry ou metrics service est active

### Modifications
- CPU limit augmente de 200m a 800m
- Memory limit augmente de 128Mi a 256Mi
- NOTES.txt enrichi avec infos existingSecret, metrics et NetworkPolicy
- Message d'erreur du token clarifie quand `existingSecret` n'est pas defini

## [0.2.0]

### Ajouts
- Deployment avec probes liveness/readiness
- Secret pour le tunnel token
- ServiceAccount avec automountServiceAccountToken: false
- PodDisruptionBudget
- Security context complet (runAsNonRoot, readOnlyRootFilesystem, drop ALL, seccomp)
- Rolling update zero downtime
- Checksum/secret annotation pour redemarrage auto
