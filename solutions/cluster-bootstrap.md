## Step 4 - initialize the master

No, the `master` node is not ready because the kubelet is not ready.
Reason: `runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady message:docker: network plugin is not ready: cni config uninitialized``

For this reason, the CoreDNS pods cannot be scheduled on the master because of the `node.kubernetes.io/not-ready` taint.

# CoreDNS

## Services

From default namespace:
* nslookup kube-dns: KO
* nslookup kube-dns.kube-system: OK
* nslookup kube-dns.kube-system.svc: OK
* nslookup kube-dns.kube-system.svc.${CLUSTER_DOMAIN}: OK

-> /etc/resolv.conf appends the search domains `default.svc.cluster.local svc.cluster.local cluster.local`

From kube-system namespace:
* nslookup kube-dns: OK
* nslookup kube-dns.kube-system: OK
* nslookup kube-dns.kube-system.svc: OK
* nslookup kube-dns.kube-system.svc.${CLUSTER_DOMAIN}: OK

-> /etc/resolv.conf appends the search domains `kube-system.svc.cluster.local svc.cluster.local cluster.local`

## Pods

From default namespace:
* nslookup W-X-Y-Z: KO
* nslookup W-X-Y-Z.kube-system: KO
* nslookup W-X-Y-Z.kube-system.pod: KO
* nslookup W-X-Y-Z.kube-system.pod.${CLUSTER_DOMAIN}: OK

-> /etc/resolv.conf appends the search domains `default.svc.cluster.local svc.cluster.local cluster.local`

From kube-system namespace:
* nslookup W-X-Y-Z: KO
* nslookup W-X-Y-Z.kube-system: KO
* nslookup W-X-Y-Z.kube-system.pod: OK
* nslookup W-X-Y-Z.kube-system.pod.${CLUSTER_DOMAIN}: OK

-> /etc/resolv.conf appends the search domains `kube-system.svc.cluster.local svc.cluster.local cluster.local`
