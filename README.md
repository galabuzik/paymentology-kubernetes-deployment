# Delivery System K8s Manifests

This repo contains Kubernetes manifest files for deploying the microservices developed in the previous task. The Docker containers were built in the previous task as well, and the Dockerfiles are available in the respective directories of each service.

 The repo also includes kubernetes manifest files required to deploy the observability stack on Kubernetes.
## Overview


- **Two core business services**: Orders Service and Delivery Service.
- **Full observability stack**: OpenTelemetry, Grafana, Prometheus/Mimir, Tempo, and Loki.
- **Auto-scaling capabilities**: Horizontal Pod Autoscaling (HPA)
- **High availability**: Pod Disruption Budgets and Load Balancing.
- **Flexible deployment**: Both Helm charts and direct YAML manifests.

##  Components

### Core Services
- **Orders Service**: Handles order processing and management (Port 6000)
- **Delivery Service**: Manages delivery operations (Port 6001)
- **Redis**: In-memory data store.

### Observability Stack (OTEL Infrastructure)
The platform implements a complete OpenTelemetry-based observability solution:

#### OpenTelemetry Collector (`otel-collector.yml`)
- **Function**: Central telemetry data collection and routing hub
- **Protocols**: HTTP (4318) and gRPC (4317) OTLP endpoints
- **Processing**: Batches and routes metrics, traces, and logs to appropriate backends
- **Integration**: Receives data from instrumented applications and forwards to Grafana stack

#### Monitoring & Visualization
- **Grafana**: Unified dashboard and visualization platform (Port 3000)
- **Peometheus/Mimir**: High-performance metrics storage and querying
- **Tempo**: Distributed tracing backend for request tracking
- **Loki**: Log aggregation and searching system

### Deployment Options

#### 1. Helm Chart Deployment (`application-helm-chart/`)
- **Chart.yaml**: Helm chart metadata and version information
- **Templates**: Templated Kubernetes manifests for flexible deployments
- **values.yaml**: Configuration values with  defaults

#### 2. Direct YAML Deployment
Individual service manifests for direct `kubectl apply`:
- `orders-service.yaml` - Orders service deployment and service
- `delivery-service.yml` - Delivery service deployment and service

##  Configuration Files

| File | Purpose |
|------|---------|
| `ingress-setup.yaml` | AWS ALB ingress with path-based routing (`/orders`, `/deliveries`, `/grafana`) |
| `horizontal-pod-autoscaler.yml` | Auto-scaling based on CPU (70-75%) and memory (80-85%) utilization |
| `pod-disruption-budget.yml` | Ensures minimum availability during maintenance and updates |
| `grafana-stack.yml` | Complete observability backend (Grafana, Prometheus/Mimir, Tempo, Loki) |
| `redis.yml` | Redis deployment for data storage |


### Deployment Options

#### Option 1: Helm Deployment
```bash
# Deploy using Helm
helm install delivery-system ./application-helm-chart/
```

#### Option 2: Direct YAML Deployment
```bash
# Deploy infrastructure
kubectl apply -f grafana-stack.yml
kubectl apply -f otel-collector.yml
kubectl apply -f redis.yml

# Deploy services
kubectl apply -f orders-service.yaml
kubectl apply -f delivery-service.yml

# Configure scaling and availability
kubectl apply -f horizontal-pod-autoscaler.yml
kubectl apply -f pod-disruption-budget.yml

# Setup ingress
kubectl apply -f ingress-setup.yaml
```

## Accessing Services

After deployment, services are available through the ALB ingress:
- **Grafana Dashboard**: `http://<alb-dns>/grafana`
- **Orders API**: `http://<alb-dns>/orders`
- **Delivery API**: `http://<alb-dns>/deliveries`

## Notes

- Services use AWS ECR for container images
- Anonymous authentication enabled for Grafana (demo setup)
- OTEL endpoints configured for internal cluster communication
- All services run with ClusterIP for internal communication
- External access managed through ALB ingress