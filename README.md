# skupper-http-loadbalancing

![alt text](Skupperhttploadbalancing.png?raw=true)

## Pre-requisites

    1. Two Kubernetes/Openshift Clusters
    2. Skupper Cli installed on your local

### On Terminal for Cluster1 -- Setting up Skupper 

```
export KUBECONFIG=~/.kube/cluster1Kube-config
oc login to cluster1
oc new-project hello-world
skupper init
skupper token create cluster1-token
```

### On a new Terminal for Cluster2 -- Setting up skupper and linking it with cluster1

```
export KUBECONFIG=~/.kube/cluster2Kube-config
oc login to cluster2
oc new-project hello-world
skupper init
skupper link create cluster1-token --name Cluster2-link-to-cluster1
skupper link status
```

### Install the required applications on cluster1

#### Navigate to terminal for cluster1

```
kubectl create deployment hello-world-frontend --image quay.io/skupper/hello-world-frontend
kubectl create deployment hello-world-backend --image quay.io/skupper/hello-world-backend

```

### Install the required applications on cluster2

#### Navigate to terminal for cluster2

```
kubectl create deployment hello-world-frontend --image quay.io/skupper/hello-world-frontend
kubectl create deployment hello-world-backend --image quay.io/skupper/hello-world-backend

```

### Expose a skupper Service on Cluster1 and see it being visible on Cluster2

#### On Cluster1 Terminal
```
oc expose deployment/hello-world-frontend --port 8080  # expose frontend as a openshift Service 
skupper expose deployment/hello-world-backend  # expose backend as a skupper Service 
skupper service status
oc expose service/hello-world-frontend  # create a route for frontend service to make it visible outside the openshift cluster
oc get route hello-world-frontend -o jsonpath='{.spec.host}' # retrieve the route url for frontend app
```

#### On Cluster2 Terminal
```
oc expose deployment/hello-world-frontend --port 8080  # expose frontend as a openshift Service 
skupper expose deployment/hello-world-backend  # expose backend as a skupper Service 
skupper service status
oc expose service/hello-world-frontend  # create a route for frontend service to make it visible outside the openshift cluster
oc get route hello-world-frontend -o jsonpath='{.spec.host}' # retrieve the route url for frontend app
```

### Open a new Terminal window

```
export KUBECONFIG=~/.kube/cluster1Kube-config
while true; do curl `oc get route hello-world-frontend -o jsonpath='{.spec.host}'`; done
```
Observe the response coming from cluster1 backend pods

### Open a new Terminal window

```
export KUBECONFIG=~/.kube/cluster2Kube-config
while true; do curl `oc get route hello-world-frontend -o jsonpath='{.spec.host}'`; done
```
Observe the response coming from cluster2 backend pods

### On cluster2 terminal

```
skupper unexpose deployment/hello-world-backend  # expose backend as a skupper Service
```
We have now unexposed the backend service from skupper on cluster2, but it is still exposed from cluster1 and is synced back to cluster2. So The backend response should now be coming from cluster1 instead of cluster2. Check the response in the terminal window where cluster2 curl is running.



