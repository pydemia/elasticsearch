# ELK stack in Kubernetes

## Installation

https://github.com/elastic/helm-charts/blob/master/elasticsearch/README.md

* [Elasticsearch](https://github.com/elastic/helm-charts/blob/master/elasticsearch/README.md)
* [LogStash](https://github.com/elastic/helm-charts/blob/master/logstash/README.md)
* [Kibana](https://github.com/elastic/helm-charts/blob/master/kibana/README.md)

```bash
helm repo add elastic https://helm.elastic.co

NAMESPACE="elasticsearch"
kubectl create namespace ${NAMESPACE}

helm install elasticsearch elastic/elasticsearch \
  --namespace ${NAMESPACE} \
  --set image=docker.elastic.co/elasticsearch/elasticsearch-oss \
  --set networkPolicy.http.enabled=true \
  > install_elasticsearch.log
helm install logstash elastic/logstash \
  --namespace ${NAMESPACE} \
  --set image=docker.elastic.co/logstash/logstash-oss \
  > install_logstash.log
helm install kibana elastic/kibana \
  --namespace ${NAMESPACE} \
  --set image=docker.elastic.co/kibana/kibana-oss \
  > install_kibana.log
```

## Set Host in `fluent-bit`

```bash
kubectl -n logging apply -f - <<EOF
apiVersion: v1
kind: Service
metadata:
  name: elasticsearch-master
spec:
  type: ExternalName
  externalName: elasticsearch-master.elasticsearch.svc.cluster.local
  ports:
  - port: 9200
    targetPort: 9200
    protocol: TCP
EOF
```

```bash
kubectl port-forward -n ${NAMESPACE} svc/elasticsearch-master 9200

curl localhost:9200/_cat/indices
```

```bash
kubectl port-forward -n ${NAMESPACE} svc/kibana-kibana 5601

curl localhost:5601/_cat/indices
```

## dejavu

https://github.com/appbaseio/dejavu

```bash
docker pull appbaseio/dejavu
docker run -p 1358:1358 -d appbaseio/dejavu
```

```bash
kubectl -n elasticsearch apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dejavu-web
  labels:
    app: dejavu
spec:
  replicas: 1
  selector:
    matchLabels:
      app: dejavu
  template:
    metadata:
      labels:
        app: dejavu
    spec:
      containers:
      - name: dejavu
        image: appbaseio/dejavu:v3.4.6
        ports:
        - containerPort: 1358
---
apiVersion: v1
kind: Service
metadata:
  name: dejavu-web
  labels:
    app: dejavu
spec:
  ports:
  - port: 1358
    protocol: TCP
  selector:
    app: dejavu
EOF
```

```bash
kubectl -n elasticsearch apply -f - <<EOF
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: elasticsearch
  namespace: elasticsearch
spec:
  hosts:
  # - "*.airuntime.com"
  # - "*.prd.airuntime.com"
  - es.airuntime.com
  - es.dev.airuntime.com
  - es.prd.airuntime.com
  - es.stg.airuntime.com
  gateways:
  - ingress-prd.istio-system
  - ingress-istio-prd.istio-system
  http:
  # - headers:
  #     request:
  #       add:
  #         Authorization: "Bearer "
  - route:
    - destination:
        host: elasticsearch-master.elasticsearch.svc.cluster.local
        port:
          number: 9200
    # rewrite:
    #   uri: /
  # tls:
  # - match:
  #   - port: 443
  #     sniHosts:
  #     - dashboard.airuntime.com
  #     - dashboard.dev.airuntime.com
  #     - dashboard.prd.airuntime.com
  #     - dashboard.stg.airuntime.com
  #   route:
  #   - destination:
  #       host: kubernetes-dashboard.kubernetes-dashboard.svc.cluster.local
  #       port: 
  #         number: 8443
---
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: elasticsearch
  namespace: elasticsearch
  # namespace: kubernetes-dashboard
spec:
  host: elasticsearch
  trafficPolicy:
    tls:
      mode: DISABLE
---
EOF
```