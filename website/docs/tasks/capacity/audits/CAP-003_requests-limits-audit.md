# CAP-003 — Requests/limits audit

- Status: Completed
- Dependencies: CAP-001 — Cluster capacity and saturation dashboard

## Context

Ensuring every workload has CPU and memory requests/limits is critical for scheduler fairness, predictable performance, and accurate capacity planning.

## Method

- Used `kubectl` with explicit kubeconfig to inventory deployments cluster-wide and surface resource requests/limits.
- Commands used:

```bash
kubectl get deployments --all-namespaces \
  --kubeconfig=$HOME/.kube/config \
  -o=custom-columns=NAMESPACE:.metadata.namespace,NAME:.metadata.name,CPU_REQUESTS:.spec.template.spec.containers[*].resources.requests.cpu,CPU_LIMITS:.spec.template.spec.containers[*].resources.limits.cpu,MEMORY_REQUESTS:.spec.template.spec.containers[*].resources.requests.memory,MEMORY_LIMITS:.spec.template.spec.containers[*].resources.limits.memory | cat
```

## Findings

The following deployments were found missing requests and/or limits at the time of audit:

- argocd: `argocd-applicationset-controller`, `argocd-dex-server`, `argocd-notifications-controller`, `argocd-redis`, `argocd-repo-server`, `argocd-server`
- calico-apiserver: `calico-apiserver`
- calico-system: `calico-kube-controllers`, `calico-typha`, `goldmane`, `whisker`
- cert-manager: `cert-manager`, `cert-manager-cainjector`, `cert-manager-webhook`
- ingress: `ingress-nginx-controller` (missing limits)
- kube-system: `cluster-autoscaler-aws-cluster-autoscaler`, `coredns` (missing cpu limit), `metrics-server` (missing limits)
- kyverno: `kyverno-admission-controller`, `kyverno-background-controller`, `kyverno-cleanup-controller`, `kyverno-reports-controller` (all missing cpu limits)
- prometheus: `blackbox-exporter-prometheus-blackbox-exporter`, `prometheus-grafana`, `prometheus-kube-prometheus-operator`, `prometheus-kube-state-metrics`

Cross-reference: detailed write-up in `docs/docs/capacity-audit.md`.

## Recommendations

- Add baseline requests/limits for all listed controllers and workloads; prioritize control-plane-adjacent and ingress paths first (ingress-nginx, coredns, cert-manager, argocd).
- For app workloads, set requests ≈ p95 observed usage and limits at a safe multiple (e.g., 1.5–2x) pending VPA.
- Consider enabling a Kyverno policy to enforce presence of requests/limits once baseline is applied.

## Links

- Documentation: `docs/docs/capacity-audit.md`
- Task log pointer: `tasks/task_log.md`
- Related: `tasks/48_capacity_baseline.md`

## Acceptance

- Inventory exists in this file and in `docs/docs/capacity-audit.md`.
- Short pointer added to `tasks/task_log.md` linking here and to the documentation.
- Follow-up issues to set requests/limits are tracked separately.
