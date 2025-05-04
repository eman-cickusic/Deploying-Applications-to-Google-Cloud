# Deploying Applications to Google Cloud

This repository provides a step-by-step guide to deploying applications to different Google Cloud services:
- App Engine
- Kubernetes Engine
- Cloud Run

## Project Overview

This project demonstrates how to deploy a simple Python Flask application to three different Google Cloud services. Each deployment method has its own advantages and use cases:

- **App Engine**: Fully managed platform that makes deployment simple with minimal configuration
- **Kubernetes Engine**: Container orchestration platform for more complex applications that need scaling
- **Cloud Run**: Serverless platform for containerized applications with automatic scaling

## Prerequisites

- Google Cloud Platform account
- Google Cloud SDK installed
- Docker installed (for Kubernetes Engine and Cloud Run deployments)
- Basic knowledge of Python and Flask

## Demo Video

I've created a video demonstration of myself showing all the steps involved in deploying the application to each service. 

https://youtu.be/wgl3bDEznag?si=3hzakoj17tAv029E

## Application Structure

The application is a simple Python Flask web application that displays a welcome message.

```
.
├── Dockerfile           # Docker configuration file
├── app.yaml             # App Engine configuration file
├── kubernetes-config.yaml  # Kubernetes configuration file
├── main.py              # Main application file
├── requirements.txt     # Python dependencies
└── templates/           # HTML templates
    ├── index.html       # Index page template
    └── layout.html      # Layout template
```

## Deployment Instructions

### 1. App Engine Deployment

App Engine is Google's fully managed platform for building and deploying applications.

1. Create `app.yaml` file:
```yaml
runtime: python39
```

2. Create the App Engine application:
```bash
gcloud app create --region=REGION_NAME
```

3. Deploy the application:
```bash
gcloud app deploy --version=one --quiet
```

4. To deploy a new version without directing traffic to it (for testing):
```bash
gcloud app deploy --version=two --no-promote --quiet
```

5. Access your application at: `https://PROJECT_ID.appspot.com`

### 2. Kubernetes Engine Deployment

Kubernetes Engine is Google's managed Kubernetes service.

1. Create a Kubernetes cluster:
```bash
gcloud container clusters create CLUSTER_NAME --zone=ZONE
```

2. Connect to your cluster:
```bash
gcloud container clusters get-credentials CLUSTER_NAME --zone=ZONE
```

3. Build your Docker image and push it to Artifact Registry:
```bash
# Create repository
gcloud artifacts repositories create devops-demo \
    --repository-format=docker \
    --location=REGION

# Configure Docker authentication
gcloud auth configure-docker REGION-docker.pkg.dev

# Build and push your image
gcloud builds submit --tag REGION-docker.pkg.dev/PROJECT_ID/devops-demo/devops-image:v0.2 .
```

4. Create `kubernetes-config.yaml` with your image path:
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: devops-deployment
  labels:
    app: devops
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: devops
      tier: frontend
  template:
    metadata:
      labels:
        app: devops
        tier: frontend
    spec:
      containers:
      - name: devops-demo
        image: REGION-docker.pkg.dev/PROJECT_ID/devops-demo/devops-image:v0.2
        ports:
        - containerPort: 8080

---
apiVersion: v1
kind: Service
metadata:
  name: devops-deployment-lb
  labels:
    app: devops
    tier: frontend-lb
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: devops
    tier: frontend
```

5. Deploy to Kubernetes:
```bash
kubectl apply -f kubernetes-config.yaml
```

6. Check your pods and service:
```bash
kubectl get pods
kubectl get services
```

7. Access your application via the external IP of the load balancer service.

### 3. Cloud Run Deployment

Cloud Run is a fully managed platform for containerized applications.

1. Build your Docker image and push it to Artifact Registry:
```bash
gcloud builds submit --tag REGION-docker.pkg.dev/PROJECT_ID/devops-demo/cloud-run-image:v0.1 .
```

2. Deploy to Cloud Run:
```bash
gcloud run deploy hello-cloud-run \
  --image=REGION-docker.pkg.dev/PROJECT_ID/devops-demo/cloud-run-image:v0.1 \
  --platform=managed \
  --region=REGION \
  --allow-unauthenticated \
  --max-instances=6
```

3. Access your application via the URL provided in the command output.


## Comparison of Deployment Methods

| Feature | App Engine | Kubernetes Engine | Cloud Run |
|---------|-----------|-------------------|-----------|
| Setup complexity | Low | High | Medium |
| Configuration | Simple app.yaml | Kubernetes YAML files | Minimal |
| Scaling | Automatic | Manual/Automatic | Automatic |
| Pricing | Pay for resources used | Pay for cluster | Pay per use |
| Use case | Simple web applications | Complex applications | Stateless containers |

## Resources

- [Google Cloud App Engine Documentation](https://cloud.google.com/appengine/docs)
- [Google Kubernetes Engine Documentation](https://cloud.google.com/kubernetes-engine/docs)
- [Google Cloud Run Documentation](https://cloud.google.com/run/docs)
