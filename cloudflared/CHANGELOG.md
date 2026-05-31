# Changelog

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
