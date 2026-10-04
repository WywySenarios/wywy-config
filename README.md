# wywy-config

Single source of truth for Wywy's cluster-wide configuration.

## Contents

- `wywy.yml` — the raw config (tables, columns, datatypes, pipelines).
- `kustomization.yaml` — self-renders `wywy.yml` into a ConfigMap named
  `wywy-config` (key `wywy.yml`, no hash suffix).

## How it is consumed

- **Cluster**: each consumer's ArgoCD app adds this repo as a _multi-source_
  (`repoURL: https://github.com/WywySenarios/wywy-config.git`, `targetRevision: <tag>`,
  `path: .`). ArgoCD renders the `wywy-config` ConfigMap into that consumer's
  namespace. Consumers mount it at `/config.yml` (`subPath: wywy.yml`).
- **Tests**: consumer repos submodule this repo pinned to the same tag and build
  the ConfigMap from `wywy-config/wywy.yml`.

## Versioning

Tags are immutable config versions (`v1.0.0`, ...). The consumer submodule and the
ArgoCD `targetRevision` must pin the **same tag**, so an old consumer checkout
resolves the exact config that was current when it was built.

- Never force-push or rewrite history.
- Never delete or retag a published tag.

## Consumer rule (single-owner)

Service repos must **never render** `wywy-config` themselves — they reference it by
name only. The cluster renders it via ArgoCD multi-source; tests render it from the
submodule.

## Render check

```sh
kubectl kustomize .
```

Should emit a `ConfigMap/wywy-config` whose `data.wywy.yml` matches `wywy.yml`.
