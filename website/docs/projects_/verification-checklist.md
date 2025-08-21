---
sidebar_position: 17
sidebar_label: 'Verification Checklist'
---

# Verification Checklist

This checklist should be used to verify the system after a fresh deployment.

- [ ] The frontend is accessible at its domain.
- [ ] The backend is accessible at its `/api` endpoint.
- [ ] The `/api/health` endpoint returns 200.
- [ ] The Argo CD UI is accessible.
- [ ] The Grafana UI is accessible.
- [ ] Logs are flowing into Loki.
- [ ] Metrics are flowing into Prometheus.
- [ ] Alerts are configured in Alertmanager.
- [ ] The staging and prod ClusterIssuers are Ready.
- [ ] A test certificate can be issued for the staging domain.
- [ ] DNS records are automatically created for Ingress resources.
- [ ] The default deny network policy is in place.
- [ ] The frontend can communicate with the backend.
- [ ] Pods are running as non-root.
- [ ] Resource requests and limits are set on all pods.
- [ ] Images are from an allowed registry.
- [ ] Image signatures are being verified.
