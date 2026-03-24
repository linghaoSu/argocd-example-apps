# Issue 26954 Reproduction

This example reproduces Argo CD issue [#26954](https://github.com/argoproj/argo-cd/issues/26954):

- parent app: App of Apps
- child app: explicitly sets `spec.syncPolicy.automated.prune: false`
- affected version from the report: Argo CD `3.3.4`

The example is intentionally minimal:

- [parent-application.yaml](parent-application.yaml) is applied once to create the parent app.
- [children/child-application.yaml](children/child-application.yaml) is the only resource managed by the parent app.
- the child app points to the existing `guestbook` example in this repository.

## Reproduce

Apply the parent application:

```bash
kubectl apply -f issue-26954/parent-application.yaml
```

Sync the parent app:

```bash
argocd app sync issue-26954-parent
argocd app wait issue-26954-parent --operation --health --timeout 120
```

On Argo CD `3.3.4`, the child application is created, but the parent app does not stay `Synced`. It returns to `OutOfSync` because the child app explicitly contains `prune: false`.

You can confirm the behavior with:

```bash
argocd app get issue-26954-parent
argocd app diff issue-26954-parent
kubectl get application issue-26954-child -n argocd -o yaml
```

The diff should be isolated to the child `Application` spec, specifically the explicit `spec.syncPolicy.automated.prune: false` setting.

## Cleanup

```bash
kubectl delete -f issue-26954/parent-application.yaml
kubectl delete application -n argocd issue-26954-child --ignore-not-found
```
