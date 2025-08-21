---
id: slos
slug: /reference/slos
sidebar_position: 22
---

# SLOs and Error Budgets

:::info This document defines the Service Level Objectives (SLOs) and error budget policy for this project. :::

## What are SLOs and Error Budgets?

- **SLO (Service Level Objective)**: A target value or range of values for a service level that is measured by a service level indicator (SLI).
- **Error Budget**: The amount of time that a service can fail to meet its SLO without consequence.

## Service Level Objectives

### Frontend

- **Availability**: 99.9% of requests will be successful (return a 2xx or 3xx status code).
- **Latency**: 95% of requests will be served in under 200ms.

### Backend

- **Availability**: 99.95% of requests will be successful.
- **Latency**: 99% of requests will be served in under 100ms.

:::warning Error Budget Policy If the error budget is consumed, all new feature development will be paused until the service is brought back into compliance with its SLOs. The team will focus on reliability and stability improvements until the error budget is replenished. :::
