# OpenSearch

## Install

```bash
helm repo add opensearch https://opensearch-project.github.io/helm-charts/
helm repo update
helm search repo opensearch
```

```bash
helm pull opensearch/opensearch \
  --version "2.12.0" \
   --untar

helm pull opensearch/opensearch-dashboards \
  --version "2.10.0" \
  --untar
```

```bash
helm upgrade --install opensearch \
  ./opensearch \
  -n opensearch \
  --create-namespace \
  -f opensearch-values.yaml

helm upgrade --install opensearch-dashboards \
  ./opensearch-dashboards \
  -n opensearch \
  --create-namespace \
  -f opensearch-dashboards-values.yaml
```