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
- 01-MySQL-ExternalName    
- 02-akv-provider.yaml
- 03-webapp-deployment.yaml
- 04-webapp-service.yaml
- externalDNS.yaml
- ingress.yaml
- cert-issuer.yaml
- secrets.yaml 
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
- DB is hosted outside this AKS cluster and is connected to this cluster using external service

---

## 🌐 Network Route from End User to Webservice

This section explains the full path a request takes — from an external user accessing the application, all the way to the backend service deployed inside Azure Kubernetes Service (AKS).

### 🔁 Flow Overview:

![image](https://github.com/user-attachments/assets/c8d92f23-43fd-44e3-bbd4-7dd298ea964a)


1. **User Access via HTTPS**  
   The user accesses the application using a custom domain over HTTPS, e.g.:  
   `https://modusetpdomain.com`

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
   - Each pod is deployed and runs the containerized web application.

7. **External Service Communication**  
   - The application may call **external APIs** (e.g., payments, notifications) via secure endpoints.
   - Authentication credentials are managed via **Azure Key Vault**.
   - Used to communicate with the Azure DB for MySQL that was existing earlier

8. **Response Delivery**  
   - The backend pod processes the request and returns the response.
   - The response follows the same route in reverse, back to the user over HTTPS.
