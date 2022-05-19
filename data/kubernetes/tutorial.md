# Kubernetes for Developers

### Setup
```
# install kubectl
sudo apt-get update && sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt-get update
sudo apt-get install -y  kubectl


# install kind
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.12.0/kind-linux-amd64
chmod +x ./kind
mv ./kind /usr/local/bin
kind version


# install minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb
sudo dpkg -i minikube_latest_amd64.deb

sudo apt-get install -y conntrack

sudo minikube start --vm-driver=none


# basic commands
kind create cluster
kind create cluster --name kind-2
kind get clusters
kubectl cluster-info --context kind-kind
kubectl cluster-info --context kind-kind-2

kind delete cluster

# turn of swappiness

# install other programs
sudo apt-get install kubeadm kubectl kubelet kubernetes-cni kube*

# init 
sudo kubeadm init
```

### Container Orchestration | Run Containers at Scale
[Attach Screenshots]

kube-proxy: responsible for taking customer requests
kubelet: communicates with master node
api server: communicates to controller, scheduler and key/val store
key/val store: responsible for keeping all the app configs to ensure current state and desired state

An LB can be used to talk to control plane, it's not for application access 


### Networking
- Pods can communicate with all other nodes without NAT
- CNI (Container Network Interface) : Specification used for communication, it uses one of the implementing plugin.

#### Kubernetes Network Requirement:
- All pods can communicate with all other without NAT
- All nodes running pods can comm. with other pods without NAT
- IP that a pod sees itself as is, the same IP that other pods see it as.


### HTTP API
[Attach Screenshot]

### Pod Creation
[create pod file](pod.yml)

```
kubectl create -f pod.yml 

kubectl get pods

kubectl get pods -o wide 

k describe pod mypod
```


### Namespaces
Partition our cluster into projects
- Provides Multitenancy
eg: devNS, prodNS

```
k get namespaces

k create namespace test

k get pods -n test

k delete namespace test
```

Default cluster, user are defined under ~/.kube/config file
we can add namespace: test to tell default namespace to connect to

`In kube-system namespace kubectl's service runs`

[create pod with 3 container](pod-multicontainer.yml)

### ReplicaSet

[create replicaset](replicaset.yml)

```
k apply -f pod-replicaset.yml

k get pods

k get replicaset

k delete replicaset nginx

k get pod -o yaml

k get pod pod_name -o yaml
```

- when we do `describe` or `get pod -o yaml` we get ownerReference field as well that contains the owner name 
- If we delete a pod from replicaset, a new pod will come up to maintain desired state

```
k get pods --show-labels
```

- In replicaset, `spec.selector.matchLabels` is responsible to determine how replicas will be selected to form the ReplicaSet
- In replicaset, `spec.template` is responsible to give template of the desired pod.
- `spec.template.metadata.labels` will be used as a label

### Deployment
[create deployment](deployment.yml)

```
k apply -f deployment.yml

k get rs

k get deployment
k get deployment

k delete deploy nginx-deploy
```

`create` command can only create an object but `apply` command can do changes at the runtime

#### Scale Up/ Scale Down
update the replica count in deploy.yml and run `apply` command

or

```
k scale deployment nginx-deploy --replicas=10
```

#### Rolling Update Strategy
`main diff between deployment to replicaset`
- A deployment can have multiple replicaset
- During deployment, a new rs is created and pods move from old replicaset to new replicaset
- In autosclaing, we define some minimum number of pods and `HPA` takes care of autoscaling based on thresholds like CPU usage.
- Any update in pod config (template) will trigger the deployment.

```
k set image deployment/nginx-deploy nginx-app=nginx:stable

k rollout status deployment nginx-deploy
```

[refer rolling update config](deployment-rolling-strategy.yml)

#### Recreate Update Strategy
[recreate update config](deployment-recreate-strategy.yml)


### Working with selectors

```
k get pods --show-labels

k get pods -l tier=frontend

k get pods -l tier=frontend, release=prod

k get pods -l tier=frontend

k get pods -l tier
```

#### Set based selector
```
k get pods -l 'release in (frontend)'

k get pods -l 'release in (prod, beta)'

k get pods -l 'tier in (frontend), release notin (prod, beta)'
```

#### matchExpression
```
selector:
    matchLabels:
        component: redis
    matchExpressions:
        - {key: tier, operator: In, values: [cache]}
        - {key: environment, operator: NotIn, values: [dev]}
```

#### Field Selectors:

```
# create pod at runtime
k run nginx --image=nginx:alpine

k get pods --field-selector status.phase=Running

k get pods --all-namespaces --field-selector status.phase=Running

# create namespace
k create ns v1

k run nginx --image=nginx:alpine -n v1

k get pods --all-namespaces --field-selector=metadata.name=nginx

```

#### Edit Pod(Object) at runtime 
```
k edit pod podname
```

### Annotations:
[annotation-pod](pod-annotations.yml)
```
k apply -f pod-annotation.yml

k describe pod nginx
```

### App Health Check
 
1. Liveness Probe : kubelet on each node does the check for each container, restarts container in case of failure.
   - [liveness-http](liveness-http-pod.yml)
   - [liveness-tcp](liveness-tcp-pod.yml)
   - [liveness-custom](liveness-custom-pod.yml)
2. Readiness Probe : Checks if app is ready to take the requests, kubelet is responsible for this check. Container is not restarted in case of failure
   - [readiness-http](readiness-http-pod.yml)
3. Startup Probe: We shouldn't do readiness and liveness probe till the app starts, this is helped by startup probe


### Job



`Video 2 : 03:06:30 `