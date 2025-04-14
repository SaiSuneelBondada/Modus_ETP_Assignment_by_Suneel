# 📘 Technical Assignment – Azure DevOps Engineer Role  
**By: Naga Veera Venu Sai Suneel Bondada**  

---

## 🔷 1. Architecture Overview

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

---

## 🌐 3. Network Route (End-to-End)

```mermaid
sequenceDiagram
    participant User
    participant AppGW as Azure Application Gateway (with WAF)
    participant Ingress as NGINX Ingress Controller
    participant WebApp as Web Service Pod
    participant SQL as Azure SQL DB
    participant API as External Services (via Azure API Mgmt)

    User->>AppGW: HTTPS Request (webapp.example.com)
    AppGW->>Ingress: Forward via Internal Load Balancer
    Ingress->>WebApp: Route to web service pod
    WebApp->>SQL: Read/Write Data (via Private Link)
    WebApp->>API: Fetch Data (via Azure API Management)
