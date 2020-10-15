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
default           Active   40h
kube-node-lease   Active   40h
kube-public       Active   40h
kube-system       Active   40h
vector            Active   39s

$ kubectl get -n vector daemonset
NAME           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
vector-agent   1         1         1       1            1           <none>          58s

$ kubectl get -n vector pod
NAME                 READY   STATUS    RESTARTS   AGE
vector-agent-gv44v   1/1     Running   0          75s

$ kubectl logs -n vector daemonset/vector-agent
Oct 15 11:09:03.142  INFO vector::app: Log level "info" is enabled.
Oct 15 11:09:03.143  INFO vector::app: Loading configs. path=["/etc/vector/managed.toml", "/etc/vector/vector-agent.toml"]
Oct 15 11:09:03.145  INFO vector::sources::kubernetes_logs: obtained Kubernetes Node name to collect logs for (self) self_node_name="minikube"
Oct 15 11:09:03.151  INFO vector::topology: Running healthchecks.
Oct 15 11:09:03.151  INFO vector::topology: Starting source "kubernetes_logs"
Oct 15 11:09:03.151  INFO vector::topology: Starting sink "stdout"
Oct 15 11:09:03.151  INFO vector: Vector has started. version="0.11.0" git_version="v0.9.0-1024-g04faa5f" released="Thu, 15 Oct 2020 04:44:38 +0000" arch="x86_64"
Oct 15 11:09:03.151  INFO vector::topology::builder: Healthcheck: Passed.
Oct 15 11:09:13.408  INFO source{component_kind="source" component_name=kubernetes_logs component_type=kubernetes_logs}:file_server: vector::internal_events::file: Found new file to watch. path="/var/log/pods/kube-system_storage-provisioner_93bde4d0-9731-4785-a80e-cd27ba8ad7c2/storage-provisioner/1.log"
Oct 15 11:09:13.408  INFO source{component_kind="source" component_name=kubernetes_logs component_type=kubernetes_logs}:file_server: vector::internal_events::file: Found new file to watch. path="/var/log/pods/kube-system_storage-provisioner_93bde4d0-9731-4785-a80e-cd27ba8ad7c2/storage-provisioner/2.log"
Oct 15 11:09:13.423  INFO source{component_kind="source" component_name=kubernetes_logs component_type=kubernetes_logs}:file_server: vector::internal_events::file: Resuming to watch file. path="/var/log/pods/kube-system_coredns-f9fd979d6-sq775_a95b23c5-e9bc-4880-b586-531b40c9db0a/coredns/0.log" file_position=411
Oct 15 11:09:13.423  INFO source{component_kind="source" component_name=kubernetes_logs component_type=kubernetes_logs}:file_server: vector::internal_events::file: Found new file to watch. path="/var/log/pods/kube-system_coredns-f9fd979d6-sq775_a95b23c5-e9bc-4880-b586-531b40c9db0a/coredns/1.log"
Oct 15 11:09:13.423  INFO source{component_kind="source" component_name=kubernetes_logs component_type=kubernetes_logs}:file_server: vector::internal_events::file: Resuming to watch file. path="/var/log/pods/kube-system_kube-proxy-vxlhh_d0942b93-9877-48c9-918a-ea71d2aac57e/kube-proxy/0.log" file_position=2694
Oct 15 11:09:13.423  INFO source{component_kind="source" component_name=kubernetes_logs component_type=kubernetes_logs}:file_server: vector::internal_events::file: Found new file to watch. path="/var/log/pods/kube-system_kube-proxy-vxlhh_d0942b93-9877-48c9-918a-ea71d2aac57e/kube-proxy/1.log"
{"file":"/var/log/pods/kube-system_storage-provisioner_93bde4d0-9731-4785-a80e-cd27ba8ad7c2/storage-provisioner/1.log","kubernetes":{"container_image":"gcr.io/k8s-minikube/storage-provisioner:v3","container_name":"storage-provisioner","pod_labels":{"addonmanager.kubernetes.io/mode":"Reconcile","gcp-auth-skip-secret":"true","integration-test":"storage-provisioner"},"pod_name":"storage-provisioner","pod_namespace":"kube-system","pod_node_name":"minikube","pod_uid":"93bde4d0-9731-4785-a80e-cd27ba8ad7c2"},"message":"F1015 11:01:46.499073       1 main.go:39] error getting server version: Get \"https://10.96.0.1:443/version?timeout=32s\": dial tcp 10.96.0.1:443: connect: network is unreachable","source_type":"kubernetes_logs","stream":"stderr","timestamp":"2020-10-15T11:01:46.499555308Z"}
{"file":"/var/log/pods/kube-system_storage-provisioner_93bde4d0-9731-4785-a80e-cd27ba8ad7c2/storage-provisioner/2.log","kubernetes":{"container_image":"gcr.io/k8s-minikube/storage-provisioner:v3","container_name":"storage-provisioner","pod_labels":{"addonmanager.kubernetes.io/mode":"Reconcile","gcp-auth-skip-secret":"true","integration-test":"storage-provisioner"},"pod_name":"storage-provisioner","pod_namespace":"kube-system","pod_node_name":"minikube","pod_uid":"93bde4d0-9731-4785-a80e-cd27ba8ad7c2"},"message":"I1015 11:02:00.223249       1 leaderelection.go:242] attempting to acquire leader lease  kube-system/k8s.io-minikube-hostpath...","source_type":"kubernetes_logs","stream":"stderr","timestamp":"2020-10-15T11:02:00.223353138Z"}
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
