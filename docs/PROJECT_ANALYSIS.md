# Project Analysis

This project is a cloud-native microservices application derived from the Google Cloud "Online Boutique" sample. It is suitable as a DevOps portfolio repository because it includes:

- Multi-language microservices (Go, Node.js, Python, Java, C#)
- gRPC backend communication
- HTTP frontend
- Redis-backed state
- Kubernetes manifests, Helm chart, and Kustomize overlays
- Terraform infrastructure configuration for Google Cloud
- GitHub Actions CI workflows
- Cloud Build and Skaffold integration

## Repository Structure

- `src/` — application microservices and supporting shared files
- `protos/` — shared Protocol Buffer definitions for gRPC services
- `helm-chart/` — Helm chart for templated Kubernetes deployment
- `kustomize/` — Kustomize overlays and optional component manifests
- `kubernetes-manifests/` — basic Kubernetes manifests for deployment
- `release/` — release-ready aggregated Kubernetes manifests
- `terraform/` — Terraform infrastructure-as-code for cloud resources
- `docs/` — project documentation and guides
- `.github/workflows/` — GitHub Actions CI workflows
- `cloudbuild.yaml` — Google Cloud Build pipeline definition
- `skaffold.yaml` — Skaffold configuration for iterative development

## Architecture

The application is composed of these primary parts:

- `frontend`: HTTP web server and UI
- `productcatalogservice`: product catalog gRPC service
- `currencyservice`: currency conversion gRPC service
- `cartservice`: shopping cart gRPC service with Redis backing
- `checkoutservice`: order orchestration gRPC service
- `shippingservice`: shipping quote gRPC service
- `paymentservice`: mock payment gRPC service
- `emailservice`: order confirmation email gRPC service
- `recommendationservice`: product recommendation gRPC service
- `adservice`: ad recommendation gRPC service
- `loadgenerator`: traffic generator for load testing
- `shoppingassistantservice`: optional AI assistant service
- `redis-cart`: Redis instance for cart persistence

### Architecture Flow

```text
Browser
   |
   v
[frontend HTTP / web UI]
   |
   +--> [productcatalogservice:3550]  (product data)
   +--> [currencyservice:7000]        (currency conversion)
   +--> [cartservice:7070]            (cart storage)
   |         |
   |         +--> [redis-cart:6379]
   +--> [recommendationservice:8080]  (recommendations)
   |         +--> [productcatalogservice:3550]
   +--> [shippingservice:50051]       (shipping cost)
   +--> [adservice:9555]              (ads)
   +--> [checkoutservice:5050]        (order orchestration)
              |
              +--> [paymentservice:50051]
              +--> [emailservice:8080]
              +--> [shippingservice:50051]
              +--> [currencyservice:7000]
              +--> [cartservice:7070]
              +--> [productcatalogservice:3550]
   +--> [shoppingassistantservice:80]  (optional)
```

### Request Flow

1. A browser sends HTTP requests to the `frontend` service.
2. The `frontend` service interacts with backend services by gRPC.
3. The cart is stored in `cartservice` using Redis.
4. `checkoutservice` coordinates payment, shipping, inventory, and email.
5. `paymentservice` and `emailservice` are mock services, demonstrating external dependencies.
6. The application exposes a single front door via the frontend, while backends are isolated inside the cluster.

## Languages and Frameworks

- Go: `frontend`, `checkoutservice`, `shippingservice`, `productcatalogservice`
- Node.js: `currencyservice`, `paymentservice`
- Python: `emailservice`, `recommendationservice`, `loadgenerator`, `shoppingassistantservice`
- Java: `adservice`
- C#: `cartservice`

## APIs and Protocols

- HTTP: used by the frontend for browser access
- gRPC: used between services for service-to-service communication
- Protocol Buffers: shared contract definitions in `protos/`

## Databases and State

- Redis: `redis-cart` is used by `cartservice` for shopping cart persistence
- The project also contains hooks for Spanner and AlloyDB in the C# cart service, but the default repo uses Redis.

## Key Environment Variables

The services depend on environment variables for runtime configuration.

Common variables:

- `PORT` — service listen port override
- `ENABLE_PROFILER` / `DISABLE_PROFILER` — optional profiling integration
- `ENABLE_TRACING` / `DISABLE_TRACING` — OpenTelemetry tracing
- `COLLECTOR_SERVICE_ADDR` — OTLP collector address for tracing

Service-specific variables:

- `PRODUCT_CATALOG_SERVICE_ADDR`
- `CURRENCY_SERVICE_ADDR`
- `CART_SERVICE_ADDR`
- `RECOMMENDATION_SERVICE_ADDR`
- `SHIPPING_SERVICE_ADDR`
- `CHECKOUT_SERVICE_ADDR`
- `AD_SERVICE_ADDR`
- `SHOPPING_ASSISTANT_SERVICE_ADDR`
- `EMAIL_SERVICE_ADDR`
- `PAYMENT_SERVICE_ADDR`
- `REDIS_ADDR`
- `SPANNER_PROJECT`
- `SPANNER_CONNECTION_STRING`
- `ALLOYDB_PRIMARY_IP`

## Ports

- `frontend`: 8080 internal, 80 external via load balancer
- `productcatalogservice`: 3550
- `currencyservice`: 7000
- `cartservice`: 7070
- `checkoutservice`: 5050
- `shippingservice`: 50051
- `paymentservice`: 50051
- `emailservice`: 8080
- `recommendationservice`: 8080
- `adservice`: 9555
- `shoppingassistantservice`: 8080
- `redis-cart`: 6379

## Deployment and DevOps Tooling

- `helm-chart/` — reusable chart template for Kubernetes
- `kustomize/` — environment-specific overlays and optional components
- `release/kubernetes-manifests.yaml` — production-ready combined manifests
- `kubernetes-manifests/` — developer-friendly base manifests
- `.github/workflows/` — CI for kustomize, helm, terraform validation, and release automation
- `terraform/` — cloud infrastructure configuration
- `cloudbuild.yaml` — Cloud Build pipeline and container build definitions
- `skaffold.yaml` — local iterative build/deploy tooling

## Notes for Phase 1

- The repo currently does not contain a root `docker-compose.yml` file.
- The main Kubernetes deployment target is `release/kubernetes-manifests.yaml`.
- The Helm chart and GitHub Actions workflows are present and ready for later phases.
- This analysis document captures the project structure, architecture, services, runtime ports, environment variables, and tooling.
