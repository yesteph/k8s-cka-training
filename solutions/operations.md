# ETCD backup
Two possibilities: as root on the master or running command in the etcd-master pod

## As root on the master

On the master node, install etcd-client
```sh
sudo su -
# Install etcd-client
apt-get install -y etcd-client
ETCDCTL_API=3 etcdctl  --endpoints https://127.0.0.1:2379  snapshot save /tmp/etcd.backup --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key  /etc/kubernetes/pki/etcd/server.key
ETCDCTL_API=3 etcdctl snapshot status /tmp/etcd.backup
```

## Running command in the etcd-master pod

```sh
kubectl -n kube-system exec -ti etcd-master -- sh -c "ETCDCTL_API=3 etcdctl  --endpoints https://127.0.0.1:2379  snapshot save /var/lib/etcd/etcd.backup --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key  /etc/kubernetes/pki/etcd/server.key"
sudo cp /var/lib/etcd/etcd.backup /tmp/.
```