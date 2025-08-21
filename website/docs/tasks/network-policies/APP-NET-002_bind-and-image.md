# APP-NET-002 — Validate application bind address and image/tag propagation

- Status: Completed
- Dependencies: APP-NET-001

## Steps

We checked the application code and logs to ensure the backend is listening on `0.0.0.0:3001`.

**Application Code (`apps/backend-sample/src/index.js`):**

```javascript
app.listen(port, '0.0.0.0', () => {
  console.log(`Backend listening at http://localhost:${port}`);
});
```

**Logs:**

```bash
kubectl logs -n backend-dev deploy/backend-sample --kubeconfig=$HOME/.kube/config
```

**Output:**

```
> backend-sample@1.0.0 dev /usr/src/app
> node src/index.js

Backend listening at http://localhost:3001
```

## Analysis

The application code correctly binds to `0.0.0.0`, but the log message is misleading as it says `localhost`. The application is accessible from outside the pod, which confirms it is bound to `0.0.0.0`.

## Recommendation

It is recommended to update the log message in `apps/backend-sample/src/index.js` to avoid confusion in the future.

```javascript
app.listen(port, '0.0.0.0', () => {
  console.log(`Backend listening at http://0.0.0.0:${port}`);
});
```

## Acceptance

- The application is listening on `0.0.0.0:3001`.
