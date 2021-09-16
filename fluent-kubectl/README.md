# Be kubectl fluent

Be in the conditiions of the CKA exam: only a shell and a web-browser.

Before starting, prepare your shell:
```sh
cat << EOF > ~/.bash_aliases
alias k=kubectl
alias k=kubectl
alias kg="kubectl get"
alias kgn="kubectl get -n"
alias kgp="kubectl get pods"
alias kd="kubectl describe"
alias kdp="kubectl describe pods"
alias kns="kubectl config set-context --current --namespace"
EOF
source ~/.bash_aliases
```

## 1 Web application

### 1.1 Pods 

In the `webapplication` namespace:
* Create a Pod named `nginx-pod` using nginx:alpine image
* Create a Pod named `messenger` using redis:latest image and a label "tier=mid". Resources must be CPU req=1 and Mem req=200MiB

### 1.2 - Service creation

Create a service `messaging-service` to listen on port 80 and send traffic to the `messenger` pod on port 6379.

## 2 Deployments

Inside the namespace `web`, create a deployment `web-app` using the image 'kodekloud/webapp-color' with 2 replicas.

Expose the `web-app` application on port 30083 of each node. The web application listens on port 8080.

## 3 Create a static pod 

On the master node, create a pod named `static-pod` in the `default` namespace. It must use the `busybox:lastest` image and run `sleep 1000` at startup

## 4 Create a persistent volume

Create a PV with this file definition:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-html
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 10Mi
  hostPath:
    path: /pv/html
```

Then, create a PersistentVolumeClaim `pv-html-claim` matching the `pv-html` parameters (Storage < 10Mi and same access modes).

Once created, run a pod named `pv-html-pod` with image `nginx:latest`. This pod must mount the `pv-html-claim` PVC to the directory `/var/www/html` .

Ensure the pod is running and the PV is bound.

## 5 Pod with security context

Deploy a Pod named `timer` with a container using the `busybox:1.28` image. This container must be allowed to set the system time (Linux capability `CAP_SYS_TIME`).
The command runned by the container must be `sleep 4800`.


## 6 JSON Path 

### Iteration

Get osImage of all the nodes and put them in /tmp/nodes_os.txt.
The expected format is:
```txt
nodeName1 osImage1
nodeName2 osImage2
nodeName3 osImage3
```

### Sort

Get nodes reversly sorted by name, and output cpu capacity in the form: name,cpucapcity
