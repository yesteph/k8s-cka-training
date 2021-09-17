# Bootstrap a cluster with kubeadm

Official documentation is here [https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)

As seen, this supposes 6 steps:
1. Get nodes. Here, you have get 3 nodes: a `master` and two workers: `worker-0` and `worker-1`
2. Install CRI runtime
3. Install kubeadm, kubelet and kubectl
4. Initialize the master
5. Setup the PodNetwork with a CNI plugin
6. Each worker join the cluster

## Steps 1 to 3 have been already done.

We ask you to finalize the setup.

## Step 4 - initialize the master

Connect to the master instance:
```sh
# Add the private SSH key
ssh-add kubernetes-formation
# Use the SSH configuration file
ssh master -F provided_ssh_config 
```

Then run the command:
```sh
kubeadm init
```

This will create a set of certificates, initialize the etcd data store and configure static pods.
As a result, you should get a command to be run on each node to join the cluster. **Note this command to execute it later**.

Moreover, a kubeconfig file has been generated.

Once your shell has been configured to use this kubeconfig file, you can interact with the cluster.

```sh
# Ensure you kubectl is correctly configured
kubectl cluster-info
```

Check the nodes status as well as the pods'statuses in the `kube-system` namespace:
```sh
kubectl get nodes
kubectl get pods -n kube-system
```

Questions:
* Is the master node ready? What is the reason?
* Are **all** the pods running? Why some of them are runnning and not some others?


## Step 5 - Install Pod Network with CNI plugin

Several CNI plugins can be used. 

In our case, we choose [WeaveNet](https://www.weave.works/docs/net/latest/overview/) as it is referenced in the [official Kubernetes documentation](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/#steps-for-the-first-control-plane-node).

```sh
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

Ensure all the pods are running in kube-system namespace.

Now, the master node should become **ready**.

## Step 6 - Add worker nodes to the cluster

Connect on each worker then run the command you copied earlier:
```sh
kubeadm join 10.123.1.7:6443 --token cul8x1.fhkfml7jg5u0cek2 \
    --discovery-token-ca-cert-hash sha256:XXXX
```

At the end, verify you get a 3 node cluster.

# Static Pods

You will try to modify a static pod.

On the `master` node, look at the folder `/etc/kubernetes/manifests`.
Edit the kube-scheduler.yaml file to add a label `fromManifest:"true"`.

Is the modification applied?

# CoreDNS

Inspect the CoreDNS configuration file to retrieve the cluster domain.

Note this value `CLUSTER_DOMAIN`.

<details>

```sh
kubectl get cm coredns -n kube-system |grep kubernetes|awk '{print $2}'
```

</details>

We will perform DNS resolution.
For that, run a pod named `lookup` using `alpine` image in the `default` namespace.

Do the same thing with in the `kube-system` namespace.

<details>

```sh
kubectl run lookup --image=alpine -n default -- sh -c "sleep 1000"
kubectl run lookup --image=alpine -n kube-system -- sh -c "sleep 1000"
```

</details>

## Services

From each pod, ry to resolve the DNS `kube-dns` service:
```sh
nslookup kube-dns
nslookup kube-dns.kube-system
nslookup kube-dns.kube-system.svc
nslookup kube-dns.kube-system.svc.${CLUSTER_DOMAIN}
```

What are the successful queries? Why?

## Pods

Note the IP of a `etcd-master` pod in the `kube-system` namespace: *W.X.Y.Z*

Try to query the related DNS records:
```sh
nslookup W-X-Y-Z
nslookup W-X-Y-Z.kube-system
nslookup W-X-Y-Z.kube-system.pod
nslookup W-X-Y-Z.kube-system.pod.${CLUSTER_DOMAIN}
```

# CNI plugin

Look at the content of the `/etc/cni/net.d/` directory.

What are the CNI plugins used by the Kubelets?