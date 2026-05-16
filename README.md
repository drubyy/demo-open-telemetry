## Purpose

Demo OpenTelemetry: metrics + traces from a Go API through a collector to Jaeger, Prometheus, and OpenObserve.

**Flow:** `api` → `otel-collector` → backends (Jaeger / Prometheus / OpenObserve)

## Layout

| Path | Role |
|------|------|
| `app/` | Go API (`/ping`) |
| `docker-compose.yml` | Local stack (Compose) |
| `k8s/config/` | Collector + Prometheus config (Compose + Kubernetes) |
| `k8s/` | Base workloads (Deployment + Service per service) |
| `overlays/minikube/` | Ingress **nginx** (local) |
| `overlays/eks/` | Ingress **ALB** (EKS) |
| `kustomization.yaml` | Default entry → `overlays/minikube` |

## Docker Compose

```bash
./bin/start-compose
# or: docker compose up --build
curl http://localhost:8088/ping
```

## Kubernetes

### Local (minikube + nginx Ingress)

```bash
./bin/start-local
```

Or step by step: `minikube addons enable ingress` → `minikube start` → `eval $(minikube docker-env)` → `docker build -t demo-api:latest ./app` → `kubectl apply -k .`

Stop workloads: `./bin/stop-local` (also stops `minikube tunnel` on macOS)

**`/etc/hosts`:** `./bin/start-local` updates it automatically (sudo once when the IP changes).

| Platform | Hosts IP | Notes |
|----------|----------|--------|
| macOS / Windows, driver `docker` | `127.0.0.1` | Run `minikube tunnel` (script starts it in the background; must stay running) |
| Linux, driver `docker` / VM drivers | `minikube ip` | Usually no tunnel |

Manual example (macOS + Docker):

```text
127.0.0.1 api.observability.local jaeger.observability.local prometheus.observability.local openobserve.observability.local
```

| UI | URL |
|----|-----|
| API | http://api.observability.local/ping |
| Jaeger | http://jaeger.observability.local |
| Prometheus | http://prometheus.observability.local |
| OpenObserve | http://openobserve.observability.local/web/ |

### EKS (ALB Ingress)

Install [AWS Load Balancer Controller](https://kubernetes-sigs.github.io/aws-load-balancer-controller/), then:

```bash
kubectl apply -k overlays/eks
```

Update hosts in `overlays/eks/ingress.yaml` to your real domains and point DNS to the ALB.

### Config changes

Edit `k8s/config/otel-collector-config.yaml` or `k8s/config/prometheus.yml`, then:

```bash
kubectl apply -k .          # local
# kubectl apply -k overlays/eks   # EKS
kubectl -n observability rollout restart deployment/otel-collector deployment/prometheus
```

### Teardown

```bash
./bin/stop-local
minikube stop
```
