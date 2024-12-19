

# TaskDock - ToDo App  

**A project demonstrating containerization with Docker, orchestration with Kubernetes, and monitoring with Prometheus.**

---

## Overview  

This project focuses on deploying a Flask-based To-Do List application with a MongoDB backend. It covers containerization, orchestration, scaling, and monitoring, providing practical examples of deploying applications in Kubernetes clusters using tools like Docker, Minikube, EKS, and Prometheus.  

---

## Key Features  

- **Containerization**: Flask application and MongoDB are containerized using Docker.  
- **Orchestration**: Kubernetes is used to manage and deploy the application in clusters.  
- **Scaling and High Availability**: Demonstrates replication controllers and rolling updates for scaling and maintaining availability.  
- **Persistent Storage**: MongoDB uses persistent volumes to ensure data retention.  
- **Monitoring**: Alerts are set up using Prometheus and integrated with Slack.  

---

## Tools and Technologies  

- **Docker & Docker Compose**: For building and running containers.  
- **Kubernetes (Minikube and EKS)**: For deployment, scaling, and orchestration.  
- **Prometheus & Alertmanager**: For application monitoring and alerting.  
- **Slack**: For receiving critical notifications.  

---

## Deployment Steps  

### 1. Containerizing the Application  

A `Dockerfile` is created to build the Flask app image:  

```Dockerfile
FROM python:3.10-slim  
WORKDIR /usr/src/app  
COPY . .  
RUN pip install --no-cache-dir -r requirements.txt  
ENV FLASK_APP=app.py FLASK_RUN_HOST=0.0.0.0  
EXPOSE 3000  
CMD ["flask", "run", "--host=0.0.0.0", "--port=3000"]  
```  

Build and push the image:  

```bash
docker build -t your-dockerhub-username/flask-app:latest .  
docker push your-dockerhub-username/flask-app:latest  
```  

### 2. Running Locally with Docker Compose  

A `docker-compose.yml` file is used to spin up Flask and MongoDB containers:  

```yaml
version: '3.8'
services:
  web:
    build: .
    ports:
      - "3000:3000"
    depends_on:
      - mongodb
    environment:
      - FLASK_ENV=development
      - MONGO_HOST=mongodb
  mongodb:
    image: mongo:latest
    volumes:
      - mongo-data:/data/db
volumes:
  mongo-data:
```  

Start the application:  

```bash
docker compose up  
```  

Access the app at `http://localhost:3000`.  

---

### 3. Deploying with Kubernetes  

#### Deployments and Services  

Create YAML files for the Flask app and MongoDB.  

**Flask App Deployment**:  

```yaml
apiVersion: apps/v1  
kind: Deployment  
metadata:  
  name: flask-app  
spec:  
  replicas: 2  
  selector:  
    matchLabels:  
      app: flask-app  
  template:  
    metadata:  
      labels:  
        app: flask-app  
    spec:  
      containers:  
      - name: flask-app  
        image: your-dockerhub-username/flask-app:latest  
        ports:  
        - containerPort: 3000  
```  

**MongoDB Deployment**:  

```yaml
apiVersion: apps/v1  
kind: Deployment  
metadata:  
  name: mongodb  
spec:  
  replicas: 1  
  selector:  
    matchLabels:  
      app: mongodb  
  template:  
    metadata:  
      labels:  
        app: mongodb  
    spec:  
      containers:  
      - name: mongodb  
        image: mongo:latest  
        ports:  
        - containerPort: 27017  
        volumeMounts:  
        - name: mongo-storage  
          mountPath: /data/db  
      volumes:  
      - name: mongo-storage  
        emptyDir: {}  
```  

Apply the configurations:  

```bash
kubectl apply -f flask-app-deployment.yaml  
kubectl apply -f mongodb-deployment.yaml  
kubectl apply -f flask-app-service.yaml  
kubectl apply -f mongodb-service.yaml  
```  

Access the app using the LoadBalancer IP.  

---

### 4. Scaling and Rolling Updates  

- **Scaling**: Adjust replicas in the deployment to scale up or down.  
- **Rolling Updates**: Update the `image` field in the deployment YAML and use:  
  ```bash
  kubectl rollout restart deployment/flask-app  
  ```  

---

### 5. Monitoring and Alerts  

**Prometheus and Slack Integration**:  

- Set up Prometheus to monitor application health.  
- Configure Alertmanager to send alerts to a Slack channel.  
- Example alert rule for detecting pod crashes:  

```yaml
groups:
  - name: example
    rules:
    - alert: CrashLoopBackOff
      expr: increase(kube_pod_container_status_restarts_total[5m]) >= 1
      for: 1m
      labels:
        severity: critical
      annotations:
        summary: "Pod {{ $labels.pod }} is crashing frequently"
        description: "Pod {{ $labels.namespace }}/{{ $labels.pod }} has restarted {{ $value }} times in the last 5 minutes."
```  

Trigger an alert by creating a pod that intentionally crashes.  

---

### Final Notes  

This project demonstrates:  
1. Effective use of Docker for containerization.  
2. Kubernetes for scalable and reliable deployment.  
3. Monitoring and alerting with Prometheus and Slack integration.  

It provides a robust foundation for deploying and managing microservices in production environments.  

--- 

