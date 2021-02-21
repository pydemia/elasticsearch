# Open Distro for Elasticsearch for Kubernetes


## Installation

https://opendistro.github.io/for-elasticsearch-docs/docs/install/helm/

* `helm >= 3`

```bash
git clone https://github.com/opendistro-for-elasticsearch/opendistro-build \
  -b v1.13.0
cd opendistro-build/helm/opendistro-es/

helm package .
# Successfully packaged chart and saved it to: /home/pydemia/git/elasticsearch/opendistro/opendistro-build/helm/opendistro-es/opendistro-es-1.13.0.tgz

NAMESPACE="elasticsearch"
helm install elasticsearch opendistro-es-1.13.0.tgz \
  --namespace=${NAMESPACE} \
  --create-namespace
#   --values=customvalues.yaml


```