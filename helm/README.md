# Deploying Vector with Helm example

## Requirements

- Kubernetes Cluster
- `kubectl`
- `helm` (we'll be using v3 in this example, v2 works too)

## Add our repo to you local Helm

First, add our repo:

```shell
$ helm repo add timberio https://packages.timber.io/helm/latest
"timberio" has been added to your repositories
```

Then find a `vector` chart in it:

```shell
$ helm search repo timberio
NAME           	CHART VERSION	APP VERSION	DESCRIPTION
timberio/vector	0.10.0       	0.10.0     	A Helm chart to collect Kubernetes logs with Ve...
```

## Prepare `values.yaml` and inspect the generated template

In this example, we've prepared `values.yaml` beforehand. Take a look at it.

Then inspect the effective config that will be applied:

```shell
$ helm template vector timberio/vector --values values.yaml --namespace vector --create-namespace
... config printed ...
```

## Deploy

```shell
$ helm install vector timberio/vector --values values.yaml --namespace vector --create-namespace
NAME: vector
LAST DEPLOYED: Fri Jun 19 08:05:04 2020
NAMESPACE: vector
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

## Results

```shell
$ helm list --all-namespaces
NAME  	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART       	APP VERSION
vector	vector   	1       	2020-06-19 08:05:04.626613499 +0300 MSK	deployed	vector-0.1.0	0.10

$ kubectl get ns
NAME              STATUS   AGE
default           Active   31s
kube-node-lease   Active   33s
kube-public       Active   33s
kube-system       Active   33s
vector            Active   15s

$ kubectl get -n vector daemonset
NAME     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
vector   1         1         1       1            1           <none>          22s

$ kubectl get -n vector pod
NAME           READY   STATUS    RESTARTS   AGE
vector-pn7zw   1/1     Running   0          23s

$ kubectl logs -n vector daemonset/vector
Jun 19 05:05:15.503  INFO vector: Log level "info" is enabled.
Jun 19 05:05:15.503  INFO vector: Loading configs. path=["/etc/vector/vector.toml"]
Jun 19 05:05:15.506  INFO vector: Vector is starting. version="0.10.0" git_version="v0.9.0-286-g55827ad" released="Fri, 12 Jun 2020 06:40:00 +0000" arch="x86_64"
Jun 19 05:05:15.506  INFO vector::sources::kubernetes_logs: obtained Kubernetes Node name to collect logs for (self) self_node_name="minikube"
Jun 19 05:05:15.512  INFO vector::topology: Running healthchecks.
Jun 19 05:05:15.512  INFO vector::topology: Starting source "kubernetes_logs"
Jun 19 05:05:15.512  INFO vector::topology: Starting sink "stdout"
Jun 19 05:05:15.512  INFO vector::topology::builder: Healthcheck: Passed.
Jun 19 05:05:25.771  INFO source{name=kubernetes_logs type=kubernetes_logs}:file_server: file_source::file_server: Found file to watch. path="/var/log/pods/kube-system_kube-proxy-6pnmb_fd8e2267-597e-4f16-86b0-28d94d8b7afc/kube-proxy/0.log" file_position=0
Jun 19 05:05:25.771  INFO source{name=kubernetes_logs type=kubernetes_logs}:file_server: file_source::file_server: Found file to watch. path="/var/log/pods/kube-system_coredns-66bff467f8-k595k_bada584a-0ea1-433b-b9a0-4f0d326610b4/coredns/0.log" file_position=0
Jun 19 05:05:25.771  INFO source{name=kubernetes_logs type=kubernetes_logs}:file_server: file_source::file_server: Found file to watch. path="/var/log/pods/kube-system_coredns-66bff467f8-ps84g_cbd4955c-b91f-48a5-9b5e-ea8536f1cce6/coredns/0.log" file_position=0
Jun 19 05:05:25.771  INFO source{name=kubernetes_logs type=kubernetes_logs}:file_server: file_source::file_server: Found file to watch. path="/var/log/pods/kube-system_storage-provisioner_0d012033-0e2a-43cc-b674-b22183b3c639/storage-provisioner/0.log" file_position=0
{"file":"/var/log/pods/kube-system_kube-proxy-6pnmb_fd8e2267-597e-4f16-86b0-28d94d8b7afc/kube-proxy/0.log","kubernetes":{"pod_labels":{"controller-revision-hash":"58b47d6fd4","k8s-app":"kube-proxy","pod-template-generation":"1"},"pod_name":"kube-proxy-6pnmb","pod_namespace":"kube-system","pod_uid":"fd8e2267-597e-4f16-86b0-28d94d8b7afc"},"message":"W0619 05:05:07.785893       1 server_others.go:559] Unknown proxy mode \"\", assuming iptables proxy","source_type":"kubernetes_logs","stream":"stderr","timestamp":"2020-06-19T05:05:07.786132550Z"}
{"file":"/var/log/pods/kube-system_kube-proxy-6pnmb_fd8e2267-597e-4f16-86b0-28d94d8b7afc/kube-proxy/0.log","kubernetes":{"pod_labels":{"controller-revision-hash":"58b47d6fd4","k8s-app":"kube-proxy","pod-template-generation":"1"},"pod_name":"kube-proxy-6pnmb","pod_namespace":"kube-system","pod_uid":"fd8e2267-597e-4f16-86b0-28d94d8b7afc"},"message":"I0619 05:05:07.797370       1 node.go:136] Successfully retrieved node IP: 172.17.0.3","source_type":"kubernetes_logs","stream":"stderr","timestamp":"2020-06-19T05:05:07.797507264Z"}
{"file":"/var/log/pods/kube-system_kube-proxy-6pnmb_fd8e2267-597e-4f16-86b0-28d94d8b7afc/kube-proxy/0.log","kubernetes":{"pod_labels":{"controller-revision-hash":"58b47d6fd4","k8s-app":"kube-proxy","pod-template-generation":"1"},"pod_name":"kube-proxy-6pnmb","pod_namespace":"kube-system","pod_uid":"fd8e2267-597e-4f16-86b0-28d94d8b7afc"},"message":"I0619 05:05:07.797413       1 server_others.go:186] Using iptables Proxier.","source_type":"kubernetes_logs","stream":"stderr","timestamp":"2020-06-19T05:05:07.797536764Z"}
...
```

## Teardown

```shell
$ helm uninstall --namespace vector vector
release "vector" uninstalled
```
