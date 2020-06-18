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

## Deploy

```shell
$ kubectl apply -k .
namespace/vector-test-kustomize created
clusterrolebinding.rbac.authorization.k8s.io/vector created
configmap/vector-config created
configmap/vector-daemonset-managed-config created
daemonset.apps/vector created
```

## Results

```shell
$ kubectl get ns
NAME                    STATUS   AGE
default                 Active   86s
kube-node-lease         Active   87s
kube-public             Active   87s
kube-system             Active   87s
vector-test-kustomize   Active   53s

$ kubectl get -n vector-test-kustomize daemonset
NAME     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
vector   1         1         1       1            1           <none>          92s

$ kubectl get -n vector-test-kustomize pod
NAME           READY   STATUS    RESTARTS   AGE
vector-s5dsx   1/1     Running   0          113s

$ kubectl logs -n vector-test-kustomize daemonset/vector
Jun 18 21:32:24.658  INFO vector: Log level "info" is enabled.
Jun 18 21:32:24.658  INFO vector: Loading configs. path=["/etc/vector/managed.toml", "/etc/vector/vector.toml"]
Jun 18 21:32:24.661  INFO vector: Vector is starting. version="0.10.0" git_version="v0.9.0-286-g55827ad" released="Fri, 12 Jun 2020 06:40:00 +0000" arch="x86_64"
Jun 18 21:32:24.662  INFO vector::sources::kubernetes_logs: obtained Kubernetes Node name to collect logs for (self) self_node_name="minikube"
Jun 18 21:32:24.667  INFO vector::topology: Running healthchecks.
Jun 18 21:32:24.667  INFO vector::topology: Starting source "kubernetes_logs"
Jun 18 21:32:24.667  INFO vector::topology: Starting sink "stdout"
Jun 18 21:32:24.667  INFO vector::topology::builder: Healthcheck: Passed.
Jun 18 21:32:34.926  INFO source{name=kubernetes_logs type=kubernetes_logs}:file_server: file_source::file_server: Found file to watch. path="/var/log/pods/kube-system_storage-provisioner_0a31c08e-18d0-4205-add5-93f8f5f8a80a/storage-provisioner/0.log" file_position=0
Jun 18 21:32:34.927  INFO source{name=kubernetes_logs type=kubernetes_logs}:file_server: file_source::file_server: Found file to watch. path="/var/log/pods/kube-system_coredns-66bff467f8-5zwbw_42651258-22a1-4712-93f3-61ec20e2c989/coredns/0.log" file_position=0
Jun 18 21:32:34.928  INFO source{name=kubernetes_logs type=kubernetes_logs}:file_server: file_source::file_server: Found file to watch. path="/var/log/pods/kube-system_coredns-66bff467f8-vwhtk_cde3eaee-c8c3-4496-a00e-5b03d834e095/coredns/0.log" file_position=0
Jun 18 21:32:34.928  INFO source{name=kubernetes_logs type=kubernetes_logs}:file_server: file_source::file_server: Found file to watch. path="/var/log/pods/kube-system_kube-proxy-r4w8b_14069ca3-a7a3-4503-8aca-ac138995f415/kube-proxy/0.log" file_position=0
{"file":"/var/log/pods/kube-system_coredns-66bff467f8-5zwbw_42651258-22a1-4712-93f3-61ec20e2c989/coredns/0.log","kubernetes":{"pod_labels":{"k8s-app":"kube-dns","pod-template-hash":"66bff467f8"},"pod_name":"coredns-66bff467f8-5zwbw","pod_namespace":"kube-system","pod_uid":"42651258-22a1-4712-93f3-61ec20e2c989"},"message":".:53","source_type":"kubernetes_logs","stream":"stdout","timestamp":"2020-06-18T21:32:07.028250477Z"}
{"file":"/var/log/pods/kube-system_coredns-66bff467f8-5zwbw_42651258-22a1-4712-93f3-61ec20e2c989/coredns/0.log","kubernetes":{"pod_labels":{"k8s-app":"kube-dns","pod-template-hash":"66bff467f8"},"pod_name":"coredns-66bff467f8-5zwbw","pod_namespace":"kube-system","pod_uid":"42651258-22a1-4712-93f3-61ec20e2c989"},"message":"[INFO] plugin/reload: Running configuration MD5 = 4e235fcc3696966e76816bcd9034ebc7","source_type":"kubernetes_logs","stream":"stdout","timestamp":"2020-06-18T21:32:07.028722894Z"}
{"file":"/var/log/pods/kube-system_coredns-66bff467f8-5zwbw_42651258-22a1-4712-93f3-61ec20e2c989/coredns/0.log","kubernetes":{"pod_labels":{"k8s-app":"kube-dns","pod-template-hash":"66bff467f8"},"pod_name":"coredns-66bff467f8-5zwbw","pod_namespace":"kube-system","pod_uid":"42651258-22a1-4712-93f3-61ec20e2c989"},"message":"CoreDNS-1.6.7","source_type":"kubernetes_logs","stream":"stdout","timestamp":"2020-06-18T21:32:07.028756019Z"}
...
```

## Teardown

```shell
$ kubectl delete -k .
namespace "vector-test-kustomize" deleted
clusterrolebinding.rbac.authorization.k8s.io "vector" deleted
configmap "vector-config" deleted
configmap "vector-daemonset-managed-config" deleted
daemonset.apps "vector" deleted

```
