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
NAME                            CHART VERSION                   APP VERSION             DESCRIPTION
timberio-nightly/vector         0.11.0-nightly-2020-11-24       nightly-2020-11-24      A Helm chart for Vector observability stack
timberio-nightly/vector-agent   0.11.0-nightly-2020-11-24       nightly-2020-11-24      A Helm chart to collect Kubernetes logs with Ve...
```

As you can see, we have two charts right now: `vector` and `vector-agent`.

`vector` chart can be used to deploy all the vector roles in one helm release.
`vector` will currently deploy `vector-agent` and, in the future,
`vector-aggregator`, and will configure them to work together out of the box.

`vector-agent` (and `vector-aggregator` in the future) can be used to deploy
individual components, if you like having separate helm releases better.

In this example, we'll be using the `vector-agent` chart.

## Prepare `values.yaml` and inspect the generated template

In this example, we've prepared `values.yaml` beforehand. Take a look at it.

Then inspect the effective config that will be applied:

```shell
$ helm template vector timberio-nightly/vector-agent --devel --values values.yaml --namespace vector
... config printed ...
```

## Deploy

```shell
$ helm install vector timberio-nightly/vector-agent --devel --values values.yaml --namespace vector --create-namespace
NAME: vector
LAST DEPLOYED: Tue Nov 24 20:34:12 2020
NAMESPACE: vector
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

## Results

```shell
$ helm list --all-namespaces
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                                   APP VERSION
vector  vector          1               2020-11-24 20:34:12.391379748 +0300 MSK deployed        vector-agent-0.11.0-nightly-2020-11-24  nightly-2020-11-24

$ kubectl get ns
NAME              STATUS   AGE
default           Active   3m23s
kube-node-lease   Active   3m25s
kube-public       Active   3m25s
kube-system       Active   3m25s
vector            Active   17s

$ kubectl get -n vector daemonset
NAME           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
vector-agent   1         1         1       1            1           <none>          26s

$ kubectl get -n vector pod
NAME                 READY   STATUS    RESTARTS   AGE
vector-agent-sfwhc   1/1     Running   0          34s

$ kubectl logs -n vector daemonset/vector-agent
Nov 24 17:34:22.714  INFO vector::app: Log level is enabled. level="vector=info,codec=info,file_source=info,tower_limit=trace,rdkafka=info"
Nov 24 17:34:22.714  INFO vector::app: Loading configs. path=["/etc/vector/managed.toml"]
Nov 24 17:34:22.718  INFO vector::sources::kubernetes_logs: Obtained Kubernetes Node name to collect logs for (self). self_node_name="minikube"
Nov 24 17:34:22.730  INFO vector::topology: Running healthchecks.
Nov 24 17:34:22.730  INFO vector::topology: Starting source. name="kubernetes_logs"
Nov 24 17:34:22.730  INFO vector::topology: Starting sink. name="stdout"
Nov 24 17:34:22.730  INFO vector: Vector has started. version="0.11.0" git_version="v0.9.0-1430-g106b250" released="Tue, 24 Nov 2020 04:40:18 +0000" arch="x86_64"
Nov 24 17:34:22.730  INFO vector::app: API is disabled, enable by setting `api.enabled` to `true` and use commands like `vector top`.
Nov 24 17:34:22.730  INFO vector::topology::builder: Healthcheck: Passed.
Nov 24 17:34:22.732  INFO source{component_kind="source" component_name=kubernetes_logs component_type=kubernetes_logs}:file_server: file_source::checkpointer: Attempting to read legacy checkpoint files.
Nov 24 17:35:24.222  INFO source{component_kind="source" component_name=kubernetes_logs component_type=kubernetes_logs}:file_server: vector::internal_events::file::source: Found new file to watch. path="/var/log/pods/kube-system_kube-proxy-8crt4_bb652a29-9877-4912-a158-6523f4b2f9d3/kube-proxy/0.log"
Nov 24 17:35:24.222  INFO source{component_kind="source" component_name=kubernetes_logs component_type=kubernetes_logs}:file_server: vector::internal_events::file::source: Found new file to watch. path="/var/log/pods/kube-system_coredns-f9fd979d6-p9xzz_96f91559-e63c-4476-be44-4af79e8bfdda/coredns/0.log"
Nov 24 17:35:24.223  INFO source{component_kind="source" component_name=kubernetes_logs component_type=kubernetes_logs}:file_server: vector::internal_events::file::source: Found new file to watch. path="/var/log/pods/kube-system_storage-provisioner_94a1659d-facb-49eb-824a-7ca4d156c5e1/storage-provisioner/0.log"
Nov 24 17:35:24.224  INFO source{component_kind="source" component_name=kubernetes_logs component_type=kubernetes_logs}:file_server: vector::internal_events::file::source: Found new file to watch. path="/var/log/pods/kube-system_storage-provisioner_94a1659d-facb-49eb-824a-7ca4d156c5e1/storage-provisioner/1.log"
{"file":"/var/log/pods/kube-system_kube-proxy-8crt4_bb652a29-9877-4912-a158-6523f4b2f9d3/kube-proxy/0.log","kubernetes":{"container_image":"k8s.gcr.io/kube-proxy:v1.19.4","container_name":"kube-proxy","pod_ip":"192.168.49.2","pod_ips":["192.168.49.2"],"pod_labels":{"controller-revision-hash":"84f98f87f8","k8s-app":"kube-proxy","pod-template-generation":"1"},"pod_name":"kube-proxy-8crt4","pod_namespace":"kube-system","pod_node_name":"minikube","pod_uid":"bb652a29-9877-4912-a158-6523f4b2f9d3"},"message":"I1124 17:31:29.102504       1 node.go:136] Successfully retrieved node IP: 192.168.49.2","source_type":"kubernetes_logs","stream":"stderr","timestamp":"2020-11-24T17:31:29.103324294Z"}
{"file":"/var/log/pods/kube-system_kube-proxy-8crt4_bb652a29-9877-4912-a158-6523f4b2f9d3/kube-proxy/0.log","kubernetes":{"container_image":"k8s.gcr.io/kube-proxy:v1.19.4","container_name":"kube-proxy","pod_ip":"192.168.49.2","pod_ips":["192.168.49.2"],"pod_labels":{"controller-revision-hash":"84f98f87f8","k8s-app":"kube-proxy","pod-template-generation":"1"},"pod_name":"kube-proxy-8crt4","pod_namespace":"kube-system","pod_node_name":"minikube","pod_uid":"bb652a29-9877-4912-a158-6523f4b2f9d3"},"message":"I1124 17:31:29.102741       1 server_others.go:111] kube-proxy node IP is an IPv4 address (192.168.49.2), assume IPv4 operation","source_type":"kubernetes_logs","stream":"stderr","timestamp":"2020-11-24T17:31:29.103428150Z"}
...
```

## Teardown

```shell
$ helm uninstall --namespace vector vector
release "vector" uninstalled
```
