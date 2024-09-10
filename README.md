# Flask Application with MongoDB on Kubernetes

This README provides instructions on how to deploy the Flask application and MongoDB on a Minikube Kubernetes cluster.

## Prerequisites

- Minikube installed and running
- kubectl configured to use Minikube
- Docker installed

## Build and Push Docker Image

1. Build the Docker image:
   ```
   docker build -t your-docker-registry/flask-app:latest .
   ```

2. Push the image to your Docker registry:
   ```
   docker push your-docker-registry/flask-app:latest
   ```

   Note: Replace `your-docker-registry` with your actual Docker registry.

## Deploy to Kubernetes

1. Apply the Kubernetes YAML file:
   ```
   kubectl apply -f kubernetes.yaml
   ```

2. Wait for all pods to be running:
   ```
   kubectl get pods -w
   ```

3. Access the Flask application:
   ```
   minikube service flask-app
   ```

## DNS Resolution in Kubernetes

DNS resolution within the Kubernetes cluster for inter-pod communication is handled by CoreDNS. Each service in the cluster gets a DNS name that follows the format: `<service-name>.<namespace>.svc.cluster.local`.

In our setup, the MongoDB service can be reached by the Flask application using the hostname `mongodb`. This works because:

1. The MongoDB StatefulSet and Service are named "mongodb".
2. Both the Flask and MongoDB pods are in the same namespace.
3. Kubernetes automatically adds the service name to the DNS resolution for pods in the same namespace.

The Flask application uses the environment variable `MONGO_HOST` set to `mongodb`, which resolves to the MongoDB service's cluster IP.

## Resource Requests and Limits

Resource requests and limits in Kubernetes help manage container resource utilization:

- Requests: The minimum amount of resources that the container needs.
- Limits: The maximum amount of resources that the container can use.

In our configuration:

- Flask app and MongoDB containers:
  - Requests: 0.2 CPU cores (200m) and 250Mi of memory
  - Limits: 0.5 CPU cores (500m) and 500Mi of memory

This ensures that each container has a guaranteed minimum of resources and cannot exceed the specified maximum, promoting stability and efficient resource utilization across the cluster.

## Design Choices

1. StatefulSet for MongoDB: Used to ensure stable network identities and persistent storage, which are crucial for databases.
2. NodePort service for Flask app: Allows external access to the application for demonstration purposes.
3. HorizontalPodAutoscaler: Enables automatic scaling based on CPU usage, ensuring the application can handle varying loads efficiently.

## Testing Scenarios

To test autoscaling and database interactions:

1. Use a tool like Apache Bench to simulate high traffic:
   ```
   ab -n 10000 -c 100 http://<minikube-ip>:<node-port>/
   ```

2. Monitor the HPA and pod scaling:
   ```
   kubectl get hpa -w
   kubectl get pods -w
   ```

3. Test database interactions:
   ```
   curl -X POST -H "Content-Type: application/json" -d '{"key": "value"}' http://<minikube-ip>:<node-port>/data
   curl http://<minikube-ip>:<node-port>/data
   ```

During testing, you may encounter issues such as:
- Initial delay in pod scaling as the HPA reacts to increased load
- Potential database connection issues if MongoDB takes time to initialize

Adjust resource allocations and HPA settings as needed based on your specific requirements and observed performance.
