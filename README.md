# `ledgermem-helm-charts`

Production-grade Helm charts for self-hosting **LedgerMem** on Kubernetes.

## Charts

| Chart | Path | What it deploys |
| --- | --- | --- |
| `ledgermem` | [`charts/ledgermem`](charts/ledgermem) | Full stack: API + worker + Postgres+pgvector + Redis |
| `ledgermem-operator` | [`charts/ledgermem-operator`](charts/ledgermem-operator) | The CRD operator (see [`ledgermem-k8s-operator`](https://github.com/ledgermem/ledgermem-k8s-operator)) |

## Install

```bash
helm repo add ledgermem oci://ghcr.io/ledgermem/charts
helm install ledgermem ledgermem/ledgermem \
  --namespace ledgermem --create-namespace \
  --set apiKey.create=true \
  --set ingress.host=memory.example.com \
  --set licenseKey="$LEDGERMEM_LICENSE_KEY"
```

## Quickstart values

```yaml
# values.yaml
image:
  repository: ghcr.io/ledgermem/api
  tag: latest
  pullPolicy: IfNotPresent

replicaCount: 2

resources:
  api:
    requests: { cpu: 500m, memory: 1Gi }
    limits:   { cpu: 2,    memory: 4Gi }
  worker:
    requests: { cpu: 200m, memory: 512Mi }

postgres:
  enabled: true
  persistence:
    size: 50Gi
    storageClass: standard

redis:
  enabled: true

ingress:
  enabled: true
  className: nginx
  host: memory.example.com
  tls: true

monitoring:
  serviceMonitor: true   # Prometheus operator
  grafanaDashboards: true

backup:
  enabled: true
  schedule: "0 2 * * *"
  s3:
    bucket: my-backups
    region: us-east-1
```

## Sub-charts

The chart pulls the following as conditional sub-charts:

- `bitnami/postgresql` — toggle off to use external Postgres (set `postgres.enabled=false` and supply `externalPostgres.uri`)
- `bitnami/redis` — toggle off similarly with `externalRedis.uri`

## Production checklist

- [ ] Set `apiKey.create=false` and supply your own secret reference
- [ ] Use external managed Postgres (RDS / Cloud SQL / Aurora) in production
- [ ] Enable `monitoring.serviceMonitor` if you run kube-prometheus-stack
- [ ] Enable `backup.enabled` with off-cluster S3-compatible target
- [ ] Set `licenseKey` from a sealed secret, not values.yaml
- [ ] Configure NetworkPolicies (`networkPolicies.enabled=true`)
- [ ] Set `podDisruptionBudget.enabled=true` on multi-replica clusters

## Publish

Charts are published to **OCI registry** at `oci://ghcr.io/ledgermem/charts` via CI on tag.

## License

Apache 2.0
