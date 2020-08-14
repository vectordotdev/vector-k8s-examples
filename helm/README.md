# Deploying Vector with Helm example

## Requirements

- Kubernetes Cluster
- `kubectl`
- `helm` (we'll be using v3 in this example, v2 works too)

## Add our repo to you local Helm

First, add our repo:

```shell
$ helm repo add timberio-nightly https://packages.timber.io/helm/nightly
"timberio-nightly" has been added to your repositories
```

> If you're using `nightly` repo - use pass `--devel` flag to `helm` commands
> when you need to perform an operation on a package from a repo, for example:
> `helm search repo timberio-nightly --devel` or
> `helm install vector timberio-nightly/vector --devel ...`.

Then find a `vector` chart in it:

```shell
$ helm search repo timberio-nightly --devel
NAME                    CHART VERSION                   APP VERSION             DESCRIPTION
timberio-nightly/vector 0.11.0-nightly-2020-08-14       nightly-2020-08-14      A Helm chart to collect Kubernetes logs with Ve...
```

## Prepare `values.yaml` and inspect the generated template

In this example, we've prepared `values.yaml` beforehand. Take a look at it.

Then inspect the effective config that will be applied:

```shell
$ helm template vector timberio-nightly/vector --devel --values values.yaml --namespace vector
... config printed ...
```

## Deploy

```shell
$ helm install vector timberio-nightly/vector --devel --values values.yaml --namespace vector --create-namespace
NAME: vector
LAST DEPLOYED: Fri Aug 14 15:50:01 2020
NAMESPACE: vector
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

## Results

```shell
$ helm list --all-namespaces
NAME  	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART                           	APP VERSION
vector	vector   	1       	2020-08-14 15:50:01.037924585 +0300 MSK	deployed	vector-0.11.0-nightly-2020-08-14	nightly-2020-08-14

$ kubectl get ns
NAME              STATUS   AGE
default           Active   4m40s
kube-node-lease   Active   4m41s
kube-public       Active   4m41s
kube-system       Active   4m41s
vector            Active   48s

$ kubectl get -n vector daemonset
NAME     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
vector   1         1         1       1            1           <none>          58s

$ kubectl get -n vector pod
NAME           READY   STATUS    RESTARTS   AGE
vector-7c8ss   1/1     Running   0          69s

$ kubectl logs -n vector daemonset/vector
Aug 14 12:50:09.951  INFO vector: Log level "info" is enabled.
Aug 14 12:50:09.952  INFO vector: Loading configs. path=["/etc/vector/vector.toml"]
Aug 14 12:50:09.954  INFO vector: Vector is starting. version="0.11.0" git_version="v0.9.0-520-g3ffc3c3" released="Fri, 14 Aug 2020 04:43:21 +0000" arch="x86_64"
Aug 14 12:50:09.954  INFO vector::sources::kubernetes_logs: obtained Kubernetes Node name to collect logs for (self) self_node_name="minikube"
Aug 14 12:50:09.959  INFO vector::topology: Running healthchecks.
Aug 14 12:50:09.959  INFO vector::topology: Starting source "kubernetes_logs"
Aug 14 12:50:09.959  INFO vector::topology: Starting sink "stdout"
Aug 14 12:50:09.959  INFO vector::topology::builder: Healthcheck: Passed.
Aug 14 12:50:20.216  INFO source{name=kubernetes_logs type=kubernetes_logs}:file_server: vector::internal_events::file: found new file to watch. path="/var/log/pods/kube-system_coredns-66bff467f8-b4kkq_d09d4c57-e25e-40e0-8e03-7cf3090e260c/coredns/0.log"
Aug 14 12:50:20.217  INFO source{name=kubernetes_logs type=kubernetes_logs}:file_server: vector::internal_events::file: found new file to watch. path="/var/log/pods/kube-system_kube-proxy-96jqt_f6de4317-54ec-491e-aa49-7e961d639486/kube-proxy/0.log"
Aug 14 12:50:20.217  INFO source{name=kubernetes_logs type=kubernetes_logs}:file_server: vector::internal_events::file: found new file to watch. path="/var/log/pods/kube-system_storage-provisioner_d39254de-c138-45a1-9a49-da66e947e948/storage-provisioner/0.log"
Aug 14 12:50:20.218  INFO source{name=kubernetes_logs type=kubernetes_logs}:file_server: vector::internal_events::file: found new file to watch. path="/var/log/pods/kube-system_storage-provisioner_d39254de-c138-45a1-9a49-da66e947e948/storage-provisioner/1.log"
{"file":"/var/log/pods/kube-system_coredns-66bff467f8-b4kkq_d09d4c57-e25e-40e0-8e03-7cf3090e260c/coredns/0.log","kubernetes":{"pod_labels":{"k8s-app":"kube-dns","pod-template-hash":"66bff467f8"},"pod_name":"coredns-66bff467f8-b4kkq","pod_namespace":"kube-system","pod_uid":"d09d4c57-e25e-40e0-8e03-7cf3090e260c"},"message":".:53","source_type":"kubernetes_logs","stream":"stdout","timestamp":"2020-08-14T12:46:22.538929177Z"}
{"file":"/var/log/pods/kube-system_coredns-66bff467f8-b4kkq_d09d4c57-e25e-40e0-8e03-7cf3090e260c/coredns/0.log","kubernetes":{"pod_labels":{"k8s-app":"kube-dns","pod-template-hash":"66bff467f8"},"pod_name":"coredns-66bff467f8-b4kkq","pod_namespace":"kube-system","pod_uid":"d09d4c57-e25e-40e0-8e03-7cf3090e260c"},"message":"[INFO] plugin/reload: Running configuration MD5 = 4e235fcc3696966e76816bcd9034ebc7","source_type":"kubernetes_logs","stream":"stdout","timestamp":"2020-08-14T12:46:22.539608097Z"}
{"file":"/var/log/pods/kube-system_coredns-66bff467f8-b4kkq_d09d4c57-e25e-40e0-8e03-7cf3090e260c/coredns/0.log","kubernetes":{"pod_labels":{"k8s-app":"kube-dns","pod-template-hash":"66bff467f8"},"pod_name":"coredns-66bff467f8-b4kkq","pod_namespace":"kube-system","pod_uid":"d09d4c57-e25e-40e0-8e03-7cf3090e260c"},"message":"CoreDNS-1.6.7","source_type":"kubernetes_logs","stream":"stdout","timestamp":"2020-08-14T12:46:22.539653686Z"}
...
```

## Teardown

```shell
$ helm uninstall --namespace vector vector
release "vector" uninstalled
```
