# 📘 Technical Assignment – Azure DevOps Engineer Role  
**By: Naga Veera Venu Sai Suneel Bondada**  

---

## 🔷 1. Architecture Overview
**The goal is to design a secure, scalable, and fault-tolerant web service infrastructure that handles:**
- Approximately 1000 requests per second (RPS).
- SQL database connectivity.
- Integration with external services.
- TLS-secured public access.

### 📌 Key Azure Components

| Layer               | Component                                          | Purpose                                                   |
|---------------------|----------------------------------------------------|-----------------------------------------------------------|
| Network Frontend    | Azure Application Gateway + WAF                   | Load balancing, SSL termination, security                 |
| Compute             | Azure Kubernetes Service (AKS)                    | Host containerized web service                            |
| Routing & TLS       | NGINX Ingress Controller + cert-manager           | URL routing and Let's Encrypt TLS                         |
| Data                | Azure SQL Database (Premium Tier)                 | Store transactional and user data                         |
| Secret Management   | Azure Key Vault / K8s Secrets                     | Store credentials and sensitive config                    |
| External APIs       | Azure API Management                              | Secure and throttle third-party integrations              |
| Monitoring          | Prometheus + Grafana                              | Resource and application monitoring                       |
| CI/CD               | Azure DevOps Pipelines                            | Automation of build, test, and deploy                     |

---

## 🏗️ 2. Infrastructure Design

### Azure Resources Used

- **AKS Cluster**  
  - Minimum 3 nodes (auto-scaling enabled)  
  - Private cluster access (optional, for secure environments)  

- **Azure SQL DB**  
  - Premium tier with geo-redundancy  

- **Application Gateway**  
  - Routes HTTPS traffic to Ingress  
  - WAF enabled for security  

- **Public IP + DNS**  
  - For accessing the service (e.g., `webapp.example.com`)  

- **Azure Key Vault**  
  - For secure storage of secrets and credentials  

- **Azure Monitor & Log Analytics Workspace**  
  - For observability and diagnostics  

- **Azure Container Registry**  
  - Stores Docker images securely  

- **Virtual Network (VNet)**  
  - With subnets for AKS, SQL, Gateway  

------

## 🔄 4. CI/CD (Azure DevOps Pipelines)

### **CI Pipeline** – Triggers on code push

- Checkout code  
- Run tests (unit, Selenium if needed)  
- Lint & scan via SonarQube  
- Build Docker image  
- Push to Azure Container Registry (ACR)  

### **CD Pipeline** – Triggers on successful build

- Pull manifests from Git repo  
- Authenticate to AKS  
- Apply deployments via `kubectl apply -f`  
- TLS via cert-manager  
- Rollout with **Blue-Green** or **Canary** strategy  

---

## 🔐 5. Security

- HTTPS via Let's Encrypt + cert-manager  
- Azure Key Vault for sensitive variables  
- Azure RBAC for AKS access control  
- Private endpoint for Azure SQL DB  
- WAF + API Management for external integrations  

---

## 📊 6. Monitoring & Alerts

Prometheus and Grafana are used for observability, deployed via Helm charts.  

**Dashboards Include:**
- CPU and memory utilization
- Requests per second (RPS)
- Pod availability and health
- Azure SQL Database performance

**Alerting:**
- Integrated with Slack, Teams, or Email via AlertManager
- Custom thresholds for latency, error rates, and availability
- Alerts for failures in external service integrations

---

## ⚡ 7. Integration with External Services

External services such as payment APIs or third-party data sources are integrated using Azure API Management (APIM).  

**Key Features:**
- Secure access via API keys stored in Azure Key Vault
- Rate limiting and throttling to prevent abuse
- Centralized API gateway for monitoring and analytics
- Retry logic and circuit breakers built into application logic

**Benefits:**
- Secured, observable, and reliable communication with external systems
- Simplified API versioning and traffic control
- Scalable access to high-volume third-party services

---

## 💥 8. High Availability & Fault Tolerance

This architecture ensures uninterrupted service and quick recovery in case of failures.

**Kubernetes Layer (AKS):**
- Multi-zone AKS cluster with auto-scaling and multiple replicas
- Health checks and liveness probes for pod resiliency

**Database Layer (Azure SQL):**
- Geo-replication and automatic failover groups
- Point-in-time restore for backup and recovery

**Routing & Networking:**
- Azure Application Gateway with WAF for secure traffic routing
- NGINX Ingress for path-based routing and load balancing

**Resiliency Practices:**
- Pod self-healing with AKS
- GitOps-style rollbacks for fast recovery
- Multi-region architecture for business continuity

---

## ✅ Summary

This solution delivers a **secure**, **highly available**, **scalable**, and **fault-tolerant** infrastructure using Microsoft Azure and modern DevOps tools.  

It supports up to **1000 requests per second**, integrates external services efficiently, and is built with continuous delivery and monitoring in mind.  

The architecture leverages services like **Azure Kubernetes Service**, **Azure DevOps**, **Ingress**, **cert-manager**, **Azure SQL**, **Azure API Management**, **Prometheus**, and **Grafana** to deliver reliable and efficient operations.



## 🌐 Network Route from End User to Webservice

This section explains the full path a request takes — from an external user accessing the application, all the way to the backend service deployed inside Azure Kubernetes Service (AKS).

### 🔁 Flow Overview:

1. **User Access via HTTPS**  
   The user accesses the application using a custom domain over HTTPS, e.g.:  
   `https://myapp.example.com`

2. **DNS Resolution**  
   - The domain is registered in Azure DNS Zones.
   - **ExternalDNS** dynamically updates DNS records to point to the **public IP** assigned to the **Ingress Controller**.

3. **SSL Termination with Let's Encrypt**  
   - **cert-manager** requests and manages SSL certificates from **Let’s Encrypt** using HTTP-01 challenge.
   - Certificates are stored as Kubernetes secrets.

4. **Ingress Controller & Load Balancing**  
   - The **Azure Load Balancer** routes incoming traffic to the **NGINX Ingress Controller**.
   - The Ingress Controller evaluates the host/path rules and forwards the request accordingly.

5. **Ingress Routing**  
   - The Ingress Controller references the Ingress resource, which defines routes to backend services (e.g., `/api`, `/web`).
   - Based on the rule, it forwards traffic to the appropriate **Kubernetes Service**.

6. **Application Pod Resolution**  
   - The **Kubernetes Service** (ClusterIP) directs the request to one of the healthy application pods.
   - Each pod is deployed via Helm and runs the containerized web application.

7. **External Service Communication**  
   - The application may call **external APIs** (e.g., payments, notifications) via secure endpoints.
   - Authentication credentials are managed via **Azure Key Vault**.

8. **Response Delivery**  
   - The backend pod processes the request and returns the response.
   - The response follows the same route in reverse, back to the user over HTTPS.


---


### 🌐 Network Route (End-to-End)

```mermaid
sequenceDiagram

    participant User
    participant AzureDNS
    participant ExternalDNS
    participant AzureLoadBalancer
    participant NGINXIngressController
    participant CertManager
    participant LetsEncrypt
    participant KubernetesSecrets
    participant IngressResource
    participant KubernetesService
    participant ApplicationPod
    participant AzureKeyVault
    participant ExternalAPI

    User->>AzureDNS: 1. HTTPS Request (myapp.example.com)
    AzureDNS-->>User: 2. Resolves to Public IP

    User->>AzureLoadBalancer: HTTPS Request to Public IP
    AzureLoadBalancer->>NGINXIngressController: Forwards Traffic

    NGINXIngressController->>CertManager: Requests Certificate (if needed)
    CertManager->>LetsEncrypt: HTTP-01 Challenge
    LetsEncrypt-->>CertManager: Challenge Response
    CertManager->>LetsEncrypt: Request Certificate
    LetsEncrypt-->>CertManager: Issues Certificate
    CertManager->>KubernetesSecrets: Stores Certificate

    NGINXIngressController->>KubernetesSecrets: Retrieves TLS Certificate
    NGINXIngressController->>User: SSL/TLS Handshake

    NGINXIngressController->>IngressResource: Evaluates Routing Rules
    IngressResource-->>NGINXIngressController: Route Configuration

    NGINXIngressController->>KubernetesService: Forwards Request (based on rules)
    KubernetesService->>ApplicationPod: Routes to a healthy pod

    ApplicationPod->>AzureKeyVault: Requests Authentication Credentials (if needed for external services)
    AzureKeyVault-->>ApplicationPod: Provides Credentials

    ApplicationPod->>ExternalAPI: Calls External API (if needed)
    ExternalAPI-->>ApplicationPod: Returns API Response

    ApplicationPod-->>KubernetesService: Returns Response
    KubernetesService-->>NGINXIngressController: Returns Response
    NGINXIngressController-->>AzureLoadBalancer: Returns Response
    AzureLoadBalancer-->>User: Returns HTTPS Response
