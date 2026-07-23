# EuroTransit

A cloud-native train-ticket marketplace built for the **Cloud Programming & Operations** capstone
(team **g03**).

## Description

EuroTransit is a Kubernetes-native train-ticket marketplace (browsing, booking, seat reservation and PayPal
payments) built to prove operational behaviour under partial failure rather than feature richness. Six
Kotlin/Spring Boot microservices behind a single Traefik ingress implement the pay-confirm-capture money path
as a fully asynchronous, orchestrated Kafka saga, with a transactional outbox drained by Debezium CDC, a
never-oversell seat-reservation invariant and idempotent exactly-once payments. The system ships via a
two-repository GitOps model continuously reconciled by Argo CD on CloudNativePG-backed, Helm-templated
services, with progressive delivery (canary and blue/green), SLOs with burn-rate alerts and a suite of chaos
experiments. A React/Vite/TypeScript storefront provides Keycloak OAuth2 (PKCE) login and live saga tracking
over SSE.

## Repositories

This is the **umbrella repository**. The system is split across two repositories, included here as git
submodules, following a two-repository **GitOps** model:

| Submodule | Half | Responsibility |
|---|---|---|
| [`capstone-application-g03`](https://github.com/Koisuji02/capstone-application-g03) | Application | Cooperating microservices (Kotlin / Spring Boot), the React storefront, tests, and CI |
| [`capstone-configuration-g03`](https://github.com/Koisuji02/capstone-configuration-g03) | Configuration | Declarative desired state: Helm charts, platform bootstrap, Argo CD apps, SLO/alert rules, and the graded `docs/` |

## Architecture at a glance

- **Six services** behind a single Traefik ingress: `catalog`, `orders`, `inventory`, `payments`,
  `notifications`, `users`.
- **Booking intake** (browse + create) is synchronous REST; the **pay → confirm → capture** money path is a
  **fully asynchronous, orchestrated Kafka command/reply saga** with `orders` as the coordinator — no
  cross-service HTTP on the money path.
- **Transactional outbox drained by Debezium CDC**; each money-path service owns a dedicated
  **CloudNativePG** PostgreSQL database.
- **Never oversell** invariant enforced by an atomic conditional `UPDATE` + DB `CHECK` (CP on the
  invariant); effective exactly-once via idempotency keys.
- **Authentication** via Keycloak (OIDC, PKCE) with a memory-only token in the SPA.

## Platform & operations

- **Kubernetes on AKS**, delivered by **GitOps**: Argo CD reconciles the cluster to the configuration repo
  (prune + self-heal); CI never holds cluster credentials.
- **Progressive delivery** with Argo Rollouts (canary + blue/green) gated on live metrics.
- **Observability**: Prometheus / Grafana RED dashboards, checkout & saga **SLOs** with multi-window
  burn-rate alerts.
- **Resilience** verified with a suite of **chaos experiments** (pod/node kills, network partitions, DB
  failover, poison messages, canary auto-abort) — see the configuration repo's `docs/`.

## Cloning

This repository uses git submodules. Clone it with:

```bash
git clone --recurse-submodules https://github.com/Koisuji02/EuroTransit.git
```

If you already cloned without `--recurse-submodules`:

```bash
git submodule update --init --recursive
```

Each submodule is pinned to a specific commit; run `git submodule update --remote` to advance them to the
latest `main`.

---

*Team capstone project (group g03). Hosted here by [Koisuji02](https://github.com/Koisuji02).*
