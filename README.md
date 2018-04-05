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
export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')
echo Name of the Pod: $POD_NAME
curl http://localhost:8001/api/v1/proxy/namespaces/default/pods/$POD_NAME/
```

## Concept
### Pods
- abstraction that represents a group of one or more containers
- Sharing:
  - Storage, as Volumes
  - Networking, as a unique cluster IP address
  - Information about how to run each container, such as the container image version or specific ports to use
