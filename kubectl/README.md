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
namespace/vector-test-kustomize created
clusterrolebinding.rbac.authorization.k8s.io/vector created
configmap/vector-agent-config created
configmap/vector-daemonset-managed-config created
daemonset.apps/vector created
```

## Results

```shell
$ kubectl get ns
NAME              STATUS   AGE
default           Active   73s
kube-node-lease   Active   75s
kube-public       Active   75s
kube-system       Active   75s
vector            Active   9s

$ kubectl get -n vector daemonset
NAME     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
vector   1         1         1       1            1           <none>          24s

$ kubectl get -n vector pod
NAME           READY   STATUS    RESTARTS   AGE
vector-x4hf5   1/1     Running   0          40s

$ kubectl logs -n vector daemonset/vector
Aug 14 12:59:40.798  INFO vector: Log level "info" is enabled.
Aug 14 12:59:40.799  INFO vector: Loading configs. path=["/etc/vector/managed.toml", "/etc/vector/vector.toml"]
Aug 14 12:59:40.803  INFO vector: Vector is starting. version="0.11.0" git_version="v0.9.0-520-g3ffc3c3" released="Fri, 14 Aug 2020 04:43:21 +0000" arch="x86_64"
Aug 14 12:59:40.803  INFO vector::sources::kubernetes_logs: obtained Kubernetes Node name to collect logs for (self) self_node_name="minikube"
Aug 14 12:59:40.811  INFO vector::topology: Running healthchecks.
Aug 14 12:59:40.811  INFO vector::topology: Starting source "kubernetes_logs"
Aug 14 12:59:40.811  INFO vector::topology: Starting sink "stdout"
Aug 14 12:59:40.811  INFO vector::topology::builder: Healthcheck: Passed.
Aug 14 12:59:51.066  INFO source{name=kubernetes_logs type=kubernetes_logs}:file_server: vector::internal_events::file: found new file to watch. path="/var/log/pods/kube-system_coredns-66bff467f8-pb4vh_f0b1e0d0-6ea8-4ae6-8d72-17b9048d15f4/coredns/0.log"
Aug 14 12:59:51.066  INFO source{name=kubernetes_logs type=kubernetes_logs}:file_server: vector::internal_events::file: found new file to watch. path="/var/log/pods/kube-system_kube-proxy-hs64v_bdd76333-e762-426c-8447-0b041cedc7f8/kube-proxy/0.log"
Aug 14 12:59:51.066  INFO source{name=kubernetes_logs type=kubernetes_logs}:file_server: vector::internal_events::file: found new file to watch. path="/var/log/pods/kube-system_storage-provisioner_f3e3a029-764a-435e-befd-c4a31fd4a96f/storage-provisioner/1.log"
Aug 14 12:59:51.066  INFO source{name=kubernetes_logs type=kubernetes_logs}:file_server: vector::internal_events::file: found new file to watch. path="/var/log/pods/kube-system_storage-provisioner_f3e3a029-764a-435e-befd-c4a31fd4a96f/storage-provisioner/2.log"
{"file":"/var/log/pods/kube-system_coredns-66bff467f8-pb4vh_f0b1e0d0-6ea8-4ae6-8d72-17b9048d15f4/coredns/0.log","kubernetes":{"pod_labels":{"k8s-app":"kube-dns","pod-template-hash":"66bff467f8"},"pod_name":"coredns-66bff467f8-pb4vh","pod_namespace":"kube-system","pod_uid":"f0b1e0d0-6ea8-4ae6-8d72-17b9048d15f4"},"message":".:53","source_type":"kubernetes_logs","stream":"stdout","timestamp":"2020-08-14T12:58:55.598884439Z"}
{"file":"/var/log/pods/kube-system_coredns-66bff467f8-pb4vh_f0b1e0d0-6ea8-4ae6-8d72-17b9048d15f4/coredns/0.log","kubernetes":{"pod_labels":{"k8s-app":"kube-dns","pod-template-hash":"66bff467f8"},"pod_name":"coredns-66bff467f8-pb4vh","pod_namespace":"kube-system","pod_uid":"f0b1e0d0-6ea8-4ae6-8d72-17b9048d15f4"},"message":"[INFO] plugin/reload: Running configuration MD5 = 4e235fcc3696966e76816bcd9034ebc7","source_type":"kubernetes_logs","stream":"stdout","timestamp":"2020-08-14T12:58:55.599356223Z"}
...
```

## Teardown

```shell
$ kubectl delete -k .
namespace "vector" deleted
clusterrolebinding.rbac.authorization.k8s.io "vector" deleted
configmap "vector-config" deleted
configmap "vector-daemonset-managed-config" deleted
daemonset.apps "vector" deleted
```
