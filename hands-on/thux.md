
## Start Minikube

```bash
minikube version
# minikube version: v0.22.3

kubectl version
# Client Version: version.Info{Major:"1", Minor:"8", GitVersion:"v1.8.1", ..
```


```bash
minikube start \
  --memory 4096
```

## Preload Docker images

To safe some time during the next steps, it is possible to load the needed Docker images upfront in Minikube:
```
minikube ssh
```
```
images="grafana/grafana:4.6.0-beta1 gcr.io/google_containers/kube-apiserver-amd64:v1.8.1 gcr.io/google_containers/kube-controller-manager-amd64:v1.8.1 gcr.io/google_containers/kube-scheduler-amd64:v1.8.1 gcr.io/google_containers/kube-proxy-amd64:v1.8.1 redis:latest quay.io/prometheus/prometheus:v1.8.0 quay.io/prometheus/node-exporter:v0.15.0 fluent/fluentd-kubernetes-daemonset:v0.12.33-elasticsearch docker.elastic.co/beats/metricbeat:6.0.0-rc1 docker.elastic.co/beats/filebeat:6.0.0-rc1 docker.elastic.co/kibana/kibana:6.0.0-rc1 docker.elastic.co/elasticsearch/elasticsearch:6.0.0-rc1 giantswarm/tiny-tools:latest busybox:latest gcr.io/google_containers/kubernetes-dashboard-amd64:v1.6.3 gcr.io/google_containers/k8s-dns-sidecar-amd64:1.14.4 gcr.io/google_containers/k8s-dns-kube-dns-amd64:1.14.4 gcr.io/google_containers/k8s-dns-dnsmasq-nanny-amd64:1.14.4 gcr.io/google-containers/kube-addon-manager:v6.4-beta.2 gcr.io/google_containers/etcd-amd64:3.0.17 marian/rebrow:latest giantswarm/thux-resolver:latest giantswarm/thux-tracker:latest dockermuenster/caddy:0.9 giantswarm/thux-frontend:latest giantswarm/thux-cleaner:latest gcr.io/google_containers/pause-amd64:3.0"

for image in $images; do
  docker pull $image
done
```
It is fine to stop Minikube after this and start later. Just don't run `minikube delete` in between.



## Dashboard

```bash
minikube dashboard
```


## Monitoring

```bash
kubectl apply \
  --filename https://raw.githubusercontent.com/giantswarm/kubernetes-training/master/hands-on/monitoring-manifests-all.yaml
```
```bash
minikube service prometheus
minikube service grafana
```

Default username/password is "admin/admin".

For the case the dashboards are missing run this to reconfigure them:
```
kubectl delete job grafana-import-dashboards
kubectl apply \
  --filename https://raw.githubusercontent.com/giantswarm/kubernetes-training/master/hands-on/monitoring-manifests-all.yaml
```

## Logging

```bash
kubectl apply \
  --filename https://raw.githubusercontent.com/giantswarm/kubernetes-training/master/hands-on/logging-manifests-all.yaml
```
```bash
minikube service elasticsearch
minikube service kibana
```

In Kibana set `fluentd-*` as `Index name or pattern`.
After that you can switch over to the `Discovery` menu item.

## Twitter Example App

```bash
kubectl create namespace thux
kubectl apply \
  --filename https://raw.githubusercontent.com/giantswarm/twitter-hot-urls-example/master/manifests-all.yaml
```

## Creating the Twitter api secrets manifest

To really bring this application up you need to get four values from your personal Twitter [account](https://twitter.com/signup) and add them to `secrets/twitter-api-secret.yaml`. For some background see the Twitter documentation about [streaming API](https://dev.twitter.com/streaming/overview/connecting).

Go to [Twitter Application Management](https://apps.twitter.com/) and create a [new](https://apps.twitter.com/app/new) application. Enter some details like these:

    Name: thux
    Description: Tracks URLs mentioned on Twitter and creates a ranked list
    Website: https://github.com/giantswarm/twitter-hot-urls-example
    Callback URL: <leave this field blank>

After that also create an Access Token under "Keys and Access Tokens". Create a file `twitter-api-secret.yaml` and fill all four data fields with the corresponding [`base64` encoded values]((http://kubernetes.io/docs/user-guide/secrets/#creating-a-secret-manually)).

```
# twitter-api-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: twitter-api
  namespace: thux
  labels:
    app: thux
type: Opaque
data:
  # values must be base64 encoded
  # with no a trailing newline.
  # use something like this:
  # printf "exampletokenxyz" | base64
  twitter-consumer-key:
  twitter-consumer-secret:
  twitter-access-token:
  twitter-access-token-secret:
```

```bash
kubectl apply --filename twitter-api-secret.yaml
```

## Scaling up and down

```bash
kubectl --namespace thux scale deployments/resolver --replicas 3

# there is an api limit. don't track too much, to pause the tracker like this:
kubectl --namespace thux scale deployments/tracker --replicas 0
```
