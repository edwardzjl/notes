# Logging System on K8S

## install

currently stuck at `tk show environments/loki` on [loki install doc](https://grafana.com/docs/loki/latest/installation/tanka/)

it seems that there's a [newer version of document](https://grafana.com/docs/loki/next/installation/tanka/) (but marked as next)

based on this doc, we need to config object storage backend

deployed by helm now.

```bash
helm upgrade --install loki --namespace=logging grafana/loki
```

As we need long time-range queries, we need to customize deploy configuration.

Or loki will throw [this error](https://github.com/grafana/loki/issues/4509)

1. Download [Default charm config](https://github.com/grafana/helm-charts/blob/main/charts/loki/values.yaml)
2. set config.limits_config.max_query_length to 0h (unlimited, the default value is 721h)
3. redeploy by helm
  ```bash
  helm upgrade --install loki --namespace=logging grafana/loki -f ${values.yaml}
  ```
