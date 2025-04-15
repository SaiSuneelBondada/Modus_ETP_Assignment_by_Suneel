# 📘 Technical Assignment – Azure DevOps Engineer Role  
**By: Naga Veera Venu Sai Suneel Bondada**  

---

## 🏛️ 1. Architecture Overview
**The goal is to design a secure, scalable, and fault-tolerant web service infrastructure that handles:**
- Approximately 1000 requests per second (RPS).
- SQL database connectivity.
- Integration with external services.
- TLS-secured public access.

### 📌 Key Azure Components

| Layer(s)                 | Component                 | Purpose                                            |
|--------------------------|---------------------------|----------------------------------------------------|
| Application, Orchestration | Azure Kubernetes Service (AKS) | Hosts and manages containerized applications.      |
| Container Registry       | Azure Container Registry  | Stores and manages Docker container images.        |
| Data                     | Azure SQL Database        | Provides persistent storage for application data. |
| Azure Application Gateway| Ingress controller        | TLS termination, and advanced routing. |
| Certifications           | Cert-manager              | Automates issuing and renewing SSL/TLS certificates. |
| Security                 | Azure Key Vault           | Securely stores and manages secrets, including certificates and database credentials. |
| Monitoring               | Azure Monitor             | Collects and analyzes logs, metrics, and provides alerting capabilities. |
| Deployment Automation    | Azure DevOps              | Enables Continuous Integration and Continuous Delivery (CI/CD) pipelines. |


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

**YAML Configs:**
- deployment.yaml – Web service deployment
- service.yaml – Kubernetes service
- ingress.yaml – Ingress + TLS
- cert-issuer.yaml – ClusterIssuer for cert-manager
- secrets.yaml – Store DB connection + API keys
- azure-pipelines.yml

------

## 🔄 3. CI/CD (Azure DevOps Pipelines)

### **CI Pipeline** – Triggers on code push

- Build Docker image  
- Push to Azure Container Registry (ACR)  

### **CD Pipeline** – Triggers on successful build

- Pull manifests from Git repo  
- Authenticate to AKS  
- Apply deployments  

---

## 🔐 4. Security

- ✅ WAF enabled on Application Gateway
- ✅ Secrets Management via Kubernetes Secrets / Azure Key Vault (AKV)
- ✅ Private networking for SQL via Service Endpoints (if desired)
- ✅ RBAC and Azure AD for access control
- ✅ Image Security with Azure Defender for Containers

---

## ⚡ 5. Integration with External Services

External APIs (e.g., payment, logistics, etc.) are integrated within the web service codebase securely using:

- API keys stored in Kubernetes Secrets or Azure Key Vault
- All outbound traffic handled from AKS Pod with firewall rules
- Resilience ensured using retry logic, circuit breakers, or API gateways (optional)

---

## 💥 6. High Availability & Fault Tolerance

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

---

## 🗒 7. Assumptions

- AKS cluster exists and is ready to use
- AKS has permissions to use the AKV

---

## ✅ Summary

This solution delivers a **secure**, **highly available**, **scalable**, and **fault-tolerant** infrastructure using Microsoft Azure and modern DevOps tools.  

It supports up to **1000 requests per second**, integrates external services efficiently, and is built with continuous delivery and monitoring in mind.  




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
