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
timberio-nightly/vector         0.11.0-nightly-2020-10-14       nightly-2020-10-14      A Helm chart for Vector observability stack
timberio-nightly/vector-agent   0.11.0-nightly-2020-10-14       nightly-2020-10-14      A Helm chart to collect Kubernetes logs with Ve...
```

As you can see, we have two charts right now: `vector` and `vector-agent`.

`vector` chart can be used to deploy all the vector roles in one helm release.
`vector` will currently deploy `vector-agent` and, in the future,
`vector-aggregator`, and will configure them to work together out of the box.

`vector-agent` (and `vector-aggregator` in the future) can be used to deploy
individual components, if you like having separate helm releases better.

In this example, we'll be using the `vector` chart. Currently, it only includes
`vector-agent` and will deploy Vector as a `DaemonSet`.

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
LAST DEPLOYED: Thu Oct 15 14:20:53 2020
NAMESPACE: vector
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

## Results

```shell
$ helm list --all-namespaces
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                               APP VERSION
vector  vector          1               2020-10-15 14:20:53.287578867 +0300 MSK deployed        vector-0.11.0-nightly-2020-10-14    nightly-2020-10-14

$ kubectl get ns
NAME              STATUS   AGE
default           Active   41h
kube-node-lease   Active   41h
kube-public       Active   41h
kube-system       Active   41h
vector            Active   41s

$ kubectl get -n vector daemonset
NAME           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
vector-agent   1         1         1       1            1           <none>          57s

$ kubectl get -n vector pod
NAME                 READY   STATUS    RESTARTS   AGE
vector-agent-vmf9k   1/1     Running   0          2m29s

$ kubectl logs -n vector daemonset/vector-agent
Oct 15 11:09:03.142  INFO vector::app: Log level "info" is enabled.
Oct 15 11:09:03.143  INFO vector::app: Loading configs. path=["/etc/vector/managed.toml"]
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
$ helm uninstall --namespace vector vector
release "vector" uninstalled
```
