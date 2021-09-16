## 1 Web application

### 1.1 Pods

Creation of the namespace and basic pod:
```sh
# Create a namespace: `webapplication`
k create ns webapplication
# Create a pod: `nginx-pod`
k run nginx-pod --image nginx:alpine -n webapplication
```

For the `messenger` pod, with must generate a yaml file then edit it:
```sh
# Generate YAML file
k run messenger --image redis:latest --labels="tier=mid" --dry-run='client' -n webapplication -o yaml > /tmp/messenger.yaml
# See documentation for resources
# k explain pod.spec
# k explain pod.spec.containers
# k explain pod.spec.containers.resources
# https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/
# Then Set resources
k edit /tmp/messenger.yaml
# Apply
k apply -f /tmp/messenger.yaml
```

### 1.2 Service creation

```sh
k expose pod messenger --port=80  --target-port=6379 -n webapplication --name=messaging-service
```

## 2 Deployments

```sh
k create ns web
k create deployment web-app --image=kodekloud/webapp-color --replicas 2 -n web
k create service nodeport web-app --node-port=30083 --tcp=80:8080 -n web
```

## 3 Create a static pod 

```sh
kubectl run static-pod --image=busybox:latest -n default --dry-run=client -o yaml  -- sleep 1000 > /etc/kubernetes/manifests/static-pod.yaml 
```

## 4 Persistent Volumes

PVC:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pv-html-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Mi
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pv-html-pod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: pv-html-claim
```

## 5 Pod with security context

```sh
# Generate Pod definition file
k run timer --image=busybox:1.28 -n default --dry-run=client -o yaml -- sleep 4800 > /tmp/pod.yaml
# View Documentation (k explain pod.spec.containers.ecurityContext)
# See https://kubernetes.io/docs/tasks/configure-pod-container/security-context/
# Then add this section: 
#   securityContext:
#     capabilities:
#       add: ["SYS_TIME"]
```

## 6 JSON Path 

### Iteration

```sh
k get nodes -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.nodeInfo.osImage}{"\n"}{end}' > /tmp/nodes_os.txt
```

### Sort

```sh
k get nodes --sort-by=.metadata.name -o jsonpath='{range .items[*]}{.metadata.name}{","}{.status.capacity.cpu}{"\n"}{end}'| | sort --reverse
```