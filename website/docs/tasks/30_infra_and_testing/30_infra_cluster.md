---
id: infra-and-cluster
slug: /tasks/30-infra-and-testing/infra-and-cluster
sidebar_position: 1
---

# Infrastructure and Cluster

### INF-001 — IaC modules skeleton

- **Dependencies:** none
- **Deliverables:** `infra/modules/cluster/{aws-eks,gcp-gke}` with variables and README; no creds/state.
- **Acceptance:** `terraform validate` passes locally with placeholders.

### INF-002 — Provision cluster

- **Dependencies:** INF-001
- **Deliverables:** instructions to provision chosen provider; private state backend configured (not committed).
- **Acceptance:** cluster reachable; `kubectl get nodes` returns ready nodes.

### INF-003 — Ingress controller

- **Dependencies:** INF-002
- **Deliverables:** install NGINX or Traefik via Helm; namespace and values.
- **Acceptance:** controller pods running and Ready.

### INF-004 — cert-manager

- **Dependencies:** INF-002
- **Deliverables:** install cert-manager; create ClusterIssuers (staging, prod) for Let’s Encrypt.
- **Acceptance:** issuers show Ready; test certificate issued for staging domain.

### INF-005 — ExternalDNS

- **Dependencies:** INF-002
- **Deliverables:** install ExternalDNS with provider credentials (configured outside repo); annotations documented.
- **Acceptance:** DNS records auto-created for Ingress resources.

### INF-006 — DNS records and domain

- **Dependencies:** INF-003, INF-005
- **Deliverables:** domain/subdomain mapped to ingress/load balancer; provider setup documented.
- **Acceptance:** domain resolves to LB IP/CNAME.

### INF-007 — Storage class (if needed)

- **Dependencies:** INF-002
- **Deliverables:** validate default storage class; document PVC behavior.
- **Acceptance:** sample PVC binds successfully.

### INF-008 — Node autoscaling (optional)

- **Dependencies:** INF-002
- **Deliverables:** enable cluster autoscaler or provider equivalent; document limits.
- **Acceptance:** scale events occur under load (observed/tested).

### INF-009 — Remote Terraform state (private)

- **Dependencies:** INF-001
- **Deliverables:** setup remote backend (S3/GCS) outside repo; note configuration steps in docs.
- **Acceptance:** `terraform init` uses remote backend; state not in git.

### INF-010 — Secrets baseline plan

- **Dependencies:** none
- **Deliverables:** document SOPS/age key management; ESO usage; secret lifecycle.
- **Acceptance:** plan reviewed; referenced by security tasks.
