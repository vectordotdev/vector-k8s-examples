# Deploying Vector with kubectl example

## Requirements

- Kubernetes Cluster
- `kubectl` >1.14

## Check the config

Take a look at `kustomization.yaml`.

Then inspect the effective config that will be applied:

```shell
$ kubectl kustomize
... config printed ...
```

> For production use, replace the `nightly-debian` tag with a specific
> version, like `nightly-2020-10-14-debian`.
> Using sliding tags, like `latest` or `nightly-debian`, is highly discouraged,
> and can lead to unexpected behaviour.
> We are only using a sliding `nightly-debian` tag instead of a more specific
> `nightly-2020-10-14-debian`-like tag in this example to avoid updating it
> for every version, but in the real world you should use a particular
> version, and keep track of updates explicitly to avoid issues.

## Deploy

```shell
$ kubectl apply -k .
namespace/vector created
serviceaccount/vector-agent created
clusterrole.rbac.authorization.k8s.io/vector-agent created
clusterrolebinding.rbac.authorization.k8s.io/vector-agent created
configmap/vector-agent-config created
configmap/vector-agent created
daemonset.apps/vector-agent created
```

## Results

```shell
$ kubectl get ns
NAME              STATUS   AGE
default           Active   28s
kube-node-lease   Active   30s
kube-public       Active   30s
kube-system       Active   30s
vector            Active   2s

$ kubectl get -n vector daemonset
NAME           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
vector-agent   1         1         1       1            1           <none>          23s

$ kubectl get -n vector pod
NAME                 READY   STATUS    RESTARTS   AGE
vector-agent-n942j   1/1     Running   0          35s

$ kubectl logs -n vector daemonset/vector-agent
Nov 24 17:20:51.098  INFO vector::app: Log level is enabled. level="info"
Nov 24 17:20:51.099  INFO vector::app: Loading configs. path=["/etc/vector/managed.toml", "/etc/vector/vector-agent.toml"]
Nov 24 17:20:51.109  INFO vector::sources::kubernetes_logs: Obtained Kubernetes Node name to collect logs for (self). self_node_name="minikube"
Nov 24 17:20:51.198  INFO vector::topology: Running healthchecks.
Nov 24 17:20:51.199  INFO vector::topology::builder: Healthcheck: Passed.
Nov 24 17:20:51.200  INFO vector::topology: Starting source. name="kubernetes_logs"
Nov 24 17:20:51.200  INFO vector::topology: Starting sink. name="stdout"
Nov 24 17:20:51.200  INFO vector: Vector has started. version="0.11.0" git_version="v0.9.0-1430-g106b250" released="Tue, 24 Nov 2020 04:40:18 +0000" arch="x86_64"
Nov 24 17:20:51.200  INFO vector::app: API is disabled, enable by setting `api.enabled` to `true` and use commands like `vector top`.
Nov 24 17:20:51.211  INFO source{component_kind="source" component_name=kubernetes_logs component_type=kubernetes_logs}:file_server: file_source::checkpointer: Attempting to read legacy checkpoint files.
Nov 24 17:21:52.719  INFO source{component_kind="source" component_name=kubernetes_logs component_type=kubernetes_logs}:file_server: vector::internal_events::file::source: Found new file to watch. path="/var/log/pods/kube-system_storage-provisioner_0a61597d-2af4-4f15-81f7-136976a9b3a4/storage-provisioner/0.log"
Nov 24 17:21:52.720  INFO source{component_kind="source" component_name=kubernetes_logs component_type=kubernetes_logs}:file_server: vector::internal_events::file::source: Found new file to watch. path="/var/log/pods/kube-system_storage-provisioner_0a61597d-2af4-4f15-81f7-136976a9b3a4/storage-provisioner/1.log"
Nov 24 17:21:52.720  INFO source{component_kind="source" component_name=kubernetes_logs component_type=kubernetes_logs}:file_server: vector::internal_events::file::source: Found new file to watch. path="/var/log/pods/kube-system_coredns-f9fd979d6-rx294_56ff8531-31e6-4d46-94ce-e0ad1f54caed/coredns/0.log"
Nov 24 17:21:52.721  INFO source{component_kind="source" component_name=kubernetes_logs component_type=kubernetes_logs}:file_server: vector::internal_events::file::source: Found new file to watch. path="/var/log/pods/kube-system_kube-proxy-nkg2f_7478c942-8e89-4a3a-94cf-b61064384658/kube-proxy/0.log"
{"file":"/var/log/pods/kube-system_storage-provisioner_0a61597d-2af4-4f15-81f7-136976a9b3a4/storage-provisioner/0.log","kubernetes":{"container_image":"gcr.io/k8s-minikube/storage-provisioner:v3","container_name":"storage-provisioner","pod_ip":"192.168.49.2","pod_ips":["192.168.49.2"],"pod_labels":{"addonmanager.kubernetes.io/mode":"Reconcile","integration-test":"storage-provisioner"},"pod_name":"storage-provisioner","pod_namespace":"kube-system","pod_node_name":"minikube","pod_uid":"0a61597d-2af4-4f15-81f7-136976a9b3a4"},"message":"F1124 17:20:21.086572       1 main.go:39] error getting server version: Get \"https://10.96.0.1:443/version?timeout=32s\": dial tcp 10.96.0.1:443: connect: network is unreachable","source_type":"kubernetes_logs","stream":"stderr","timestamp":"2020-11-24T17:20:21.087172149Z"}
{"file":"/var/log/pods/kube-system_storage-provisioner_0a61597d-2af4-4f15-81f7-136976a9b3a4/storage-provisioner/1.log","kubernetes":{"container_image":"gcr.io/k8s-minikube/storage-provisioner:v3","container_name":"storage-provisioner","pod_ip":"192.168.49.2","pod_ips":["192.168.49.2"],"pod_labels":{"addonmanager.kubernetes.io/mode":"Reconcile","integration-test":"storage-provisioner"},"pod_name":"storage-provisioner","pod_namespace":"kube-system","pod_node_name":"minikube","pod_uid":"0a61597d-2af4-4f15-81f7-136976a9b3a4"},"message":"I1124 17:20:22.001649       1 leaderelection.go:242] attempting to acquire leader lease  kube-system/k8s.io-minikube-hostpath...","source_type":"kubernetes_logs","stream":"stderr","timestamp":"2020-11-24T17:20:22.002386829Z"}
...
```

## Teardown

```shell
$ kubectl delete -k .
namespace "vector" deleted
serviceaccount "vector-agent" deleted
clusterrole.rbac.authorization.k8s.io "vector-agent" deleted
clusterrolebinding.rbac.authorization.k8s.io "vector-agent" deleted
configmap "vector-agent-config" deleted
configmap "vector-agent" deleted
daemonset.apps "vector-agent" deleted
```
