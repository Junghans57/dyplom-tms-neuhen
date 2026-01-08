<!-- <p align="center">
<img src="/src/frontend/static/icons/FinTech_Logo.svg" width="300" alt="FinTech Platform Logo" />
</p> -->
![CI Badge](https://github.com/Junghans57/fintech-transaction-platform/workflows/CI/badge.svg)


# FinTech Transaction Platform

Cloud-native microservices-based FinTech platform for processing financial transactions,
managing user accounts, and handling payments. Designed with microservices architecture
for scalability, maintainability, and full DevOps automation.

---

## Architecture

The platform consists of 11 microservices written in Go, Python, and Node.js, communicating over gRPC:

| Service | Language | Description |
| ------- | -------- | ----------- |
| **client-app (FinTech web portal)** | Go | Web client providing users access to accounts, transactions, and payment operations. |
| **accounts-service** | Python | Manages user account state and temporary transaction data during financial operations. |
| **products-service** | Go | Provides reference data such as financial products, tariffs, and service parameters. |
| **currency-exchange-service** | Node.js | Handles currency conversion and exchange rate calculations for multi-currency transactions. |
| **payment-gateway (mock)** | Node.js | Processes payment operations and simulates interaction with an external payment gateway (mock). |
| **settlement-service** | Go | Calculates fees and commissions for transaction settlements (mock). |
| **notification-service** | Python | Sends transactional notifications to users about completed operations (mock). |
| **transactions-service** | Go | Orchestrates transactions by coordinating account validation, payment processing, and notifications. |
| **fraud-check-service (mock)** | Python | Performs basic transaction analysis and simulates fraud detection and risk assessment logic. |
| **audit-service** | Python | Collects audit and contextual data related to user actions and system events. |
| **loadgenerator** | Python/Locust | Generates synthetic load simulating real user financial activity for performance and stress testing. |

---

## Deployment

The platform is deployed to a Kubernetes cluster using **Infrastructure as Code (IaC)**
and automated **CI/CD pipelines**.

### Local Development

Run the platform locally for development/testing with Docker Compose:

```bash
docker compose up



## Additional deployment options

- **Terraform**: [See these instructions](/terraform) to learn how to deploy Online Boutique using [Terraform](https://www.terraform.io/intro).
- **Istio / Cloud Service Mesh**: [See these instructions](/kustomize/components/service-mesh-istio/README.md) to deploy Online Boutique alongside an Istio-backed service mesh.
- **Non-GKE clusters (Minikube, Kind, etc)**: See the [Development guide](/docs/development-guide.md) to learn how you can deploy Online Boutique on non-GKE clusters.
- **AI assistant using Gemini**: [See these instructions](/kustomize/components/shopping-assistant/README.md) to deploy a Gemini-powered AI assistant that suggests products to purchase based on an image.
- **And more**: The [`/kustomize` directory](/kustomize) contains instructions for customizing the deployment of Online Boutique with other variations.

## Documentation

- [Development](/docs/development-guide.md) to learn how to run and develop this app locally.

## Demos featuring Online Boutique

- [Security hardening of the OnlineBoutique sample apps with the Docker Hardened Images (DHI)](https://medium.com/google-cloud/security-hardening-of-the-onlineboutique-sample-apps-with-docker-hardened-images-dhi-ca1fad348343)
- [alpine, distroless or scratch?](https://medium.com/google-cloud/alpine-distroless-or-scratch-caac35250e0b)
- [Platform Engineering in action: Deploy the Online Boutique sample apps with Score and Humanitec](https://medium.com/p/d99101001e69)
- [The new Kubernetes Gateway API with Istio and Anthos Service Mesh (ASM)](https://medium.com/p/9d64c7009cd)
- [Use Azure Redis Cache with the Online Boutique sample on AKS](https://medium.com/p/981bd98b53f8)
- [Sail Sharp, 8 tips to optimize and secure your .NET containers for Kubernetes](https://medium.com/p/c68ba253844a)
- [Deploy multi-region application with Anthos and Google cloud Spanner](https://medium.com/google-cloud/a2ea3493ed0)
- [Use Google Cloud Memorystore (Redis) with the Online Boutique sample on GKE](https://medium.com/p/82f7879a900d)
- [Use Helm to simplify the deployment of Online Boutique, with a Service Mesh, GitOps, and more!](https://medium.com/p/246119e46d53)
- [How to reduce microservices complexity with Apigee and Anthos Service Mesh](https://cloud.google.com/blog/products/application-modernization/api-management-and-service-mesh-go-together)
- [gRPC health probes with Kubernetes 1.24+](https://medium.com/p/b5bd26253a4c)
- [Use Google Cloud Spanner with the Online Boutique sample](https://medium.com/p/f7248e077339)
- [Seamlessly encrypt traffic from any apps in your Mesh to Memorystore (redis)](https://medium.com/google-cloud/64b71969318d)
- [Strengthen your app's security with Cloud Service Mesh and Anthos Config Management](https://cloud.google.com/service-mesh/docs/strengthen-app-security)
- [From edge to mesh: Exposing service mesh applications through GKE Ingress](https://cloud.google.com/architecture/exposing-service-mesh-apps-through-gke-ingress)
- [Take the first step toward SRE with Cloud Operations Sandbox](https://cloud.google.com/blog/products/operations/on-the-road-to-sre-with-cloud-operations-sandbox)
- [Deploying the Online Boutique sample application on Cloud Service Mesh](https://cloud.google.com/service-mesh/docs/onlineboutique-install-kpt)
- [Anthos Service Mesh Workshop: Lab Guide](https://codelabs.developers.google.com/codelabs/anthos-service-mesh-workshop)
- [KubeCon EU 2019 - Reinventing Networking: A Deep Dive into Istio's Multicluster Gateways - Steve Dake, Independent](https://youtu.be/-t2BfT59zJA?t=982)
- Google Cloud Next'18 SF
  - [Day 1 Keynote](https://youtu.be/vJ9OaAqfxo4?t=2416) showing GKE On-Prem
  - [Day 3 Keynote](https://youtu.be/JQPOPV_VH5w?t=815) showing Stackdriver
    APM (Tracing, Code Search, Profiler, Google Cloud Build)
  - [Introduction to Service Management with Istio](https://www.youtube.com/watch?v=wCJrdKdD6UM&feature=youtu.be&t=586)
- [Google Cloud Next'18 London – Keynote](https://youtu.be/nIq2pkNcfEI?t=3071)
  showing Stackdriver Incident Response Management
- [Microservices demo showcasing Go Micro](https://github.com/go-micro/demo)

## DevOps Scope

This project demonstrates a production-ready DevOps workflow:

- Infrastructure provisioning using Terraform
- Kubernetes-based deployment
- CI/CD automation with GitHub Actions
- Monitoring with Prometheus and Grafana
- Centralized logging with ELK stack
- Automated notifications
