# Lab 11: Docker/K3s Deployment

## Learning Objectives
- Containerize agent applications with Docker
- Deploy to Kubernetes/K3s clusters
- Implement health checks and monitoring
- Set up CI/CD pipelines
- Manage secrets and configuration

## Prerequisites
- Labs 1-10 completed
- Docker and Kubernetes basics
- Understanding of containerization

## Setup Requirements
1. Install Docker: https://docs.docker.com/get-docker/
2. Install k3d: https://k3d.io/ (for local K3s)
3. Create lab11 directory

## Step-by-Step Instructions

### Step 1: Dockerfile Creation
```dockerfile
FROM python:3.10-slim

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY . .

# Create non-root user
RUN useradd -m -u 1000 agentuser && \
    chown -R agentuser:agentuser /app
USER agentuser

# Expose port
EXPOSE 8000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD python -c "import requests; requests.get('http://localhost:8000/health')" || exit 1

# Run application
CMD ["python", "app.py"]
```

### Step 2: Docker Compose for Local Dev
```yaml
version: '3.8'

services:
  agent:
    build: .
    ports:
      - "8000:8000"
    environment:
      - OLLAMA_HOST=host.docker.internal:11434
      - LOG_LEVEL=debug
    volumes:
      - ./data:/app/data
    depends_on:
      - redis

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  ollama:
    image: ollama/ollama:latest
    ports:
      - "11434:11434"
    volumes:
      - ollama-data:/root/.ollama
```

### Step 3: Kubernetes Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: agent-deployment
  labels:
    app: agent
spec:
  replicas: 3
  selector:
    matchLabels:
      app: agent
  template:
    metadata:
      labels:
        app: agent
    spec:
      containers:
      - name: agent
        image: myregistry/agent:latest
        ports:
        - containerPort: 8000
        env:
        - name: OLLAMA_HOST
          value: "ollama-service:11434"
        - name: LOG_LEVEL
          value: "info"
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8000
          initialDelaySeconds: 5
          periodSeconds: 5
```

### Step 4: Kubernetes Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: agent-service
spec:
  selector:
    app: agent
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8000
  type: LoadBalancer
```

### Step 5: Horizontal Pod Autoscaler
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: agent-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: agent-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

### Step 6: Secrets Management
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: agent-secrets
type: Opaque
stringData:
  api-key: your-api-key-here
  database-url: postgres://user:pass@host:5432/db
```

### Step 7: CI/CD Pipeline (GitHub Actions)
```yaml
name: Deploy Agent

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Build Docker image
      run: docker build -t agent:${{ github.sha }} .
    
    - name: Run tests
      run: docker run agent:${{ github.sha }} pytest
    
    - name: Push to registry
      run: |
        echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USER }}" --password-stdin
        docker push myregistry/agent:${{ github.sha }}
    
    - name: Deploy to K8s
      run: |
        kubectl set image deployment/agent agent=myregistry/agent:${{ github.sha }}
```

## Exercises
1. **Basic**: Create Docker image and run locally
2. **Intermediate**: Deploy to K3s with HPA
3. **Advanced**: Set up full CI/CD pipeline

## Next Steps
- Lab 12: Capstone project
- Deploy production agent systems
