# Upgrade a cluster

Upgrade the cluster currently in version 1.20.9 to 1.21.3 with kubeadm.
Start with the control plane (master), then proceed to the worker.

Tips:
* Remember kubeadm and kubelet are not managed by kubeadm.
* kubeadm is tied to the kubernetes releases. You may need to update kubeadm before upgrading the cluster.
* worker nodes must be drain/uncordon

# ETCD backup

Backup the ETCD cluster and save it to /tmp/etcd.backup on the master.

Tips: you can perform the backup as a root user on the master node with the etcd-client APT package OR you can run command in etcd-master pod.

# Create an admin certificate

You will create a new cluster admin certificate using openssl:
1. Create a new certificate private key `openssl genrsa -out johndoe.key 2048`
2. Create a CSR using the previous key, and indicate the following subject: **/CN=johndoe/O=system:masters**
    * `openssl req -new -key johndoe.key -subj "/CN=johndoe/O=system:masters" -out johndoe.csr`
3. Sign the certificate. We let you retrieve the locations of the cluster CA.crt and CA.key.
    * `openssl x509 -in johndoe.csr -CA XXX -CAkey ZZZ -out johndoe.crt`
4. Test with a curl
    * `curl --insecure --cert johndoe.crt --key johndoe.key https://localhost:6443/api/v1/pods`
