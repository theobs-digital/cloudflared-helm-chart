# Changelog

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
