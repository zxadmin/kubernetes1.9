[root@k8s-node ~]# kubeadm join 192.168.1.191:6443 --token orzxom.mmqfhs7cijp1koao --discovery-token-ca-cert-hash sha256:8eb5c04ab1fc77cc76c33d09a6ac40aca527f3c7bdda6fb8e1df348d2473be99

[preflight] Running pre-flight checks.

[preflight] Some fatal errors occurred:

[ERROR CRI]: unable to check if the container runtime at "/var/run/dockershim.sock" is running: fork/exec /usr/bin/crictl -r /var/run/dockershim.sock info: no such file or directory

[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`

[root@k8s-node ~]# crictl ps

FATA[0000] listing containers failed: rpc error: code = Unavailable desc = grpc: the connection is unavailable 

[root@k8s-node ~]# crictl --version

crictl version 1.11.0

[root@k8s-node ~]# kubeadm join 192.168.1.191:6443 --token orzxom.mmqfhs7cijp1koao **--ignore-preflight-errors cri** --discovery-token-ca-cert-hash sha256:8eb5c04ab1fc77cc76c33d09a6ac40aca527f3c7bdda6fb8e1df348d2473be99

[preflight] Running pre-flight checks.

[WARNING CRI]: unable to check if the container runtime at "/var/run/dockershim.sock" is running: fork/exec /usr/bin/crictl -r /var/run/dockershim.sock info: no such file or directory

[discovery] Trying to connect to API Server "192.168.1.191:6443"

[discovery] Created cluster-info discovery client, requesting info from "https://192.168.1.191:6443"

[discovery] Requesting info from "https://192.168.1.191:6443" again to validate TLS against the pinned public key

[discovery] Cluster info signature and contents are valid and TLS certificate validates against pinned roots, will use API Server "192.168.1.191:6443"

[discovery] Successfully established connection with API Server "192.168.1.191:6443"

This node has joined the cluster:

* Certificate signing request was sent to master and a response

  was received.
  
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the master to see this node join the cluster






# 初始化失败报错

[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10255/healthz' failed with error: Get http://localhost:10255/healthz: dial tcp [::1]:10255: getsockopt: connection refused





解决办法:

cgroup driver配置要相同

查看docker cgroup driver:

docker info|grep Cgroup

有systemd和cgroupfs两种，把kubelet service配置改成与docker一致



vim /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

Environment="KUBELET_SYSTEM_PODS_ARGS=--pod-manifest-path=/etc/kubernetes/manifests --allow-privileged=true --fail-swap-on=false"

KUBELET_CGROUP_ARGS=--cgroup-driver=cgroupfs #这个配置与docker改成一致


