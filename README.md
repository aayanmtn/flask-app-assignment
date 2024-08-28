# Flask-MongoDB Kubernetes Deployment

This guide provides step-by-step instructions to deploy a Flask application that interacts with MongoDB on a Minikube Kubernetes cluster.

## Prerequisites

- Minikube installed
- kubectl installed
- Docker installed
- Docker Hub account

## Steps

### 1. Start Minikube

Start Minikube with the necessary resources:

```bash
minikube start --cpus=4 --memory=8192
kubectl apply -f mongodb-statefulset.yaml
kubectl apply -f flask-app-deployment.yaml
kubectl apply -f flask-app-hpa.yaml
minikube service flask-app-service
kubectl get hpa flask-app-hpa --watch
```
### 5. **Explanation of DNS Resolution in Kubernetes**

Kubernetes uses CoreDNS for DNS-based service discovery, allowing services to communicate using their service names. When a pod in the Flask app deployment tries to connect to MongoDB, it can simply use the DNS name `mongodb-service` (defined in the service) to resolve the IP address of the MongoDB pod. This DNS resolution facilitates easy and reliable inter-pod communication within the Kubernetes cluster.

### 6. **Explanation of Resource Requests and Limits**

- **Resource Requests**: The minimum amount of resources (CPU and memory) guaranteed to a container.
- **Resource Limits**: The maximum amount of resources a container is allowed to consume.

Setting appropriate resource requests and limits helps to ensure that critical applications have the resources they need while preventing a single container from monopolizing resources, which could destabilize the cluster.

### 7. **Design Choices**

- **StatefulSet for MongoDB**: Provides stable network identifiers, persistent storage, and ordered scaling for the MongoDB instance. Alternatives like Deployments could be used, but they don’t guarantee stable network IDs, which are crucial for stateful services like databases.
- **Horizontal Pod Autoscaler (HPA)**: Chosen to ensure the Flask application can handle traffic spikes. An alternative could be manual scaling, but HPA offers more flexibility and responsiveness to real-time traffic.

### 8. **Cookie Point: Testing Scenarios**

#### **Autoscaling Testing**

To test autoscaling, simulate high CPU load:

```bash
for i in {1..1000}; do curl http://$(minikube service flask-app-service --url)/; done
```