# hello-k8s
Personal learning notes on k8s 

## Creat a cluster
- start a minikube cluster
```bash
minikube start
```

- view the cluster/nodes details
```bash
kubectl cluster-info
kubectl get nodes
```

## Deploy an app
- creat a deployment
```bash
kubectl run NAME --image=image [--env="key=value"] [--port=port] [--replicas=replicas] [--dry-run=bool] [--overrides=inline-json] [--command] -- [COMMAND] [args...] [options]
kubectl run kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1 --port=8080
```
- list your deployments
```bash
kubectl get deployments
```
### Proxy
- creat a connection between the host and the Kubernetes cluster
```bash
kubectl proxy
curl http://localhost:8001/version
```
- access the app
```bash
curl http://localhost:8001/api/v1/proxy/namespaces/default/pods/$POD_NAME/
```
## Explore the app
- `kubectl get` - list resources
- `kubectl describe` - show detailed information about a resource
- `kubectl logs` - print the logs from a container in a pod
- `kubectl exec` - execute a command on a container in a pod

- examples:
```bash
export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')
echo Name of the Pod: $POD_NAME
kubectl logs $POD_NAME
kubectl exec $POD_NAME env
kubectl exec -ti $POD_NAME bash
```
## Exposing the app
- creat service:
```bash
kubectl get services
```
  - find out node port
```bash
kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080
kubectl describe services/kubernetes-bootcamp
export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}')
echo NODE_PORT=$NODE_PORT
```
  - access the service from external
```bash
curl $(minikube ip):$NODE_PORT
```

- using labels:
  - find out the name of label
```bash
kubectl describe deployment
```
  - find the coorsponding pods or services
```bash
kubectl get pods -l run=kubernetes-bootcamp
kubectl get services -l run=kubernetes-bootcamp
```
  - apply a new label
```bash
export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')
echo Name of the Pod: $POD_NAME
kubectl label pod $POD_NAME app=v1
kubectl describe pods $POD_NAME
kubectl get pods -l app=v1
```
  - deleting a service
```bash
kubectl delete service -l run=kubernetes-bootcamp
```
## Concept
### Pods
<div align=center><img height="50%" width="50%" src="/img/overview.png"/></div>

- abstraction that represents a group of one or more containers
- Sharing:
  - Storage, as Volumes
  - Networking, as a unique cluster IP address
  - Information about how to run each container, such as the container image version or specific ports to use
### Services
<div align=center><img height="50%" width="50%" src="/img/service.png"/></div>

- an abstraction which defines a logical set of Pods and a policy by which to access them
- can be exposed in different ways by specifying a `type` in the ServiceSpec:
  - ClusterIP (default) - Exposes the Service on an internal IP in the cluster. This type makes the Service only reachable from within the cluster.
  - NodePort - Exposes the Service on the same port of each selected Node in the cluster using NAT. Makes a Service accessible from outside the cluster using `<NodeIP>:<NodePort>`. Superset of ClusterIP.
  - LoadBalancer - Creates an external load balancer in the current cloud (if supported) and assigns a fixed, external IP to the Service. Superset of NodePort.
  - ExternalName - Exposes the Service using an arbitrary name (specified by `externalName` in the spec) by returning a CNAME record with the name. No proxy is used. This type requires v1.7 or higher of `kube-dns`.

- Services match a set of Pods using labels and selectors, a grouping primitive that allows logical operation on objects in Kubernetes. Labels are key/value pairs attached to objects and can be used in any number of ways:
  - Designate objects for development, test, and production
  - Embed version tags
  - Classify an object using tags
