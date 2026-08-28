# 🚀 Wanderlust 3-Tier Application – DevOps Project

A production-style **3-tier application deployment** implemented using modern DevOps, containerization, Kubernetes, GitOps, security scanning, and monitoring tools.

The project demonstrates an end-to-end **CI/CD + GitOps workflow** from source code commit to application deployment and monitoring on **Amazon EKS**.

---

## 🏗️ Architecture

<img width="1536" height="1024" alt="169b1788-8629-4aae-bb2f-4cb3b1f11f69" src="https://github.com/user-attachments/assets/5e23c067-c4ce-47e6-97a6-942c56fb673f" />


## 📌 Project Overview

The Wanderlust application consists of three main tiers:

- **Frontend** – React/Vite application
- **Backend** – Backend API application
- **Database** – MongoDB

The application is containerized using Docker and deployed on an **Amazon EKS Kubernetes cluster**.

The CI/CD pipeline performs:

- Source code checkout
- Code quality analysis
- Dependency vulnerability scanning
- Docker image building
- Container vulnerability scanning
- Docker image publishing
- Kubernetes manifest updates
- GitOps-based deployment
- Application monitoring

---

# 🔄 CI/CD Workflow

```text
Developer
    │
    ▼
 GitHub
    │
    ▼
 Jenkins CI
    │
    ├── SonarCloud
    │     └── Code Quality / Quality Gate
    │
    ├── OWASP Dependency-Check
    │     └── Dependency Vulnerability Scan
    │
    ├── Docker Build
    │
    └── Trivy
          └── Container Vulnerability Scan
    │
    ▼
 DockerHub
    │
    ▼
 Update Kubernetes Manifests
    │
    ▼
 GitHub
    │
    ▼
 Argo CD
    │
    ▼
 Amazon EKS
    │
    ├── Frontend
    ├── Backend
    └── Database
🛠️ Technologies Used
Category	Technology
Source Control	GitHub
CI/CD	Jenkins
Code Quality	SonarCloud
Dependency Security	OWASP Dependency-Check
Container Security	Trivy
Containerization	Docker
Container Registry	DockerHub
Orchestration	Kubernetes
Cloud	AWS
Kubernetes Platform	Amazon EKS
GitOps	Argo CD
Monitoring	Prometheus
Visualization	Grafana
Alerting	Alertmanager
📂 Project Structure
Wanderlust_3TIER/
│
├── frontend/
│   ├── Dockerfile
│   ├── package.json
│   └── ...
│
├── backend/
│   ├── Dockerfile
│   ├── package.json
│   └── ...
│
├── kubernetes/
│   ├── backend.yaml
│   ├── frontend.yaml
│   ├── database.yaml
│   └── ...
│
├── Automations/
│   ├── updatebackendnew.sh
│   └── updatefrontendnew.sh
│
└── Jenkinsfile
🔧 CI Pipeline

The Jenkins CI pipeline automates the application build and security validation process.

Pipeline Stages
1. Validate Parameters
        ↓
2. Workspace Cleanup
        ↓
3. Git Checkout
        ↓
4. OWASP Dependency Check
        ↓
5. SonarCloud Analysis
        ↓
6. Quality Gate
        ↓
7. Environment Configuration
        ↓
8. Docker Image Build
        ↓
9. Trivy Image Scan
        ↓
10. DockerHub Push
        ↓
11. Trigger CD Pipeline
🔍 SonarCloud

SonarCloud is used to analyze source code for:

Bugs
Vulnerabilities
Code smells
Maintainability issues
Security issues
Code coverage
Duplications

The Jenkins pipeline waits for the SonarCloud Quality Gate before continuing.

Jenkins
   ↓
SonarCloud Analysis
   ↓
Quality Gate
   ↓
 ┌───────────┐
 │           │
 OK        FAILED
 │           │
 ▼           ▼
Continue    Stop
🔐 Security Scanning
OWASP Dependency-Check

OWASP Dependency-Check scans application dependencies for known vulnerabilities.

Example Jenkins command:

dependencyCheck(
    additionalArguments: '--scan ./',
    odcInstallation: 'owasp'
)

The generated report is published to Jenkins.

🛡️ Trivy

Trivy is used to scan Docker images for vulnerabilities.

Example:

trivy image kevinjcloud/wanderlust:${IMAGE_TAG}

For CI security enforcement:

trivy image \
  --severity HIGH,CRITICAL \
  --exit-code 1 \
  kevinjcloud/wanderlust:${IMAGE_TAG}

If a HIGH or CRITICAL vulnerability is detected, the pipeline can fail.

🐳 Docker

Both frontend and backend application components are containerized using Docker.

Docker images are pushed to DockerHub.

Example:

kevinjcloud/wanderlust:<tag>

The image tag is passed from the CI pipeline to the CD pipeline.

☸️ Kubernetes / Amazon EKS

The application is deployed to an Amazon EKS cluster.

The Kubernetes environment contains:

EKS Cluster
│
├── Frontend Pod
│
├── Backend Pod
│
├── PostgreSQL Pod
│
├── Kubernetes Services
│
└── Monitoring Stack

The frontend application runs using Vite on port 5173.

🔄 GitOps with Argo CD

Argo CD is used for continuous deployment following the GitOps approach.

Instead of Jenkins directly deploying to Kubernetes, Jenkins updates the Kubernetes manifests in GitHub.

Example:

image: kevinjcloud/wanderlust:v1

After a new CI build:

image: kevinjcloud/wanderlust:v2

The CD pipeline commits and pushes the manifest change to GitHub.

Argo CD detects the change and synchronizes the Kubernetes cluster.

Jenkins
   │
   ▼
Update Kubernetes YAML
   │
   ▼
GitHub
   │
   ▼
Argo CD detects change
   │
   ▼
Kubernetes
   │
   ▼
New application version
📦 CD Pipeline

The CD pipeline receives the Docker image tags from the CI pipeline.

Example parameters:

FRONTEND_DOCKER_TAG
BACKEND_DOCKER_TAG

The pipeline:

1. Checkout Kubernetes repository
        ↓
2. Verify Docker image tags
        ↓
3. Update backend.yaml
        ↓
4. Update frontend.yaml
        ↓
5. Commit changes
        ↓
6. Push changes to GitHub
        ↓
7. Argo CD detects changes
        ↓
8. Application is deployed
📊 Monitoring

The application is monitored using:

Prometheus

Prometheus collects metrics from the Kubernetes environment and applications.

Grafana

Grafana is used to visualize Prometheus metrics through dashboards.

Alertmanager

Alertmanager handles alerts generated by Prometheus and can route notifications to configured channels.

Monitoring architecture:

Applications
     │
     │ Metrics
     ▼
 Prometheus
     │
     ▼
 Grafana
     │
     ▼
Alertmanager
     │
     ├── Email
     ├── Slack
     └── Other notification channels
🌐 Application Access

The Kubernetes application can be exposed using Kubernetes Services.

For example:

apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  type: LoadBalancer
  selector:
    app: frontend
  ports:
    - port: 80
      targetPort: 5173

AWS provisions a Load Balancer for external access.

Internet
   │
   ▼
AWS Load Balancer
   │
   ▼
Kubernetes Service
   │
   ▼
Frontend Pod
   │
   ▼
Vite :5173
🚀 Deployment Flow

The complete deployment flow is:

                 ┌─────────────┐
                 │  Developer  │
                 └──────┬──────┘
                        │
                        ▼
                 ┌─────────────┐
                 │   GitHub    │
                 └──────┬──────┘
                        │
                        ▼
                 ┌─────────────┐
                 │   Jenkins   │
                 └──────┬──────┘
                        │
          ┌─────────────┼──────────────┐
          ▼             ▼              ▼
     SonarCloud       OWASP          Trivy
          │             │              │
          └─────────────┼──────────────┘
                        ▼
                 ┌─────────────┐
                 │    Docker   │
                 │    Build    │
                 └──────┬──────┘
                        │
                        ▼
                 ┌─────────────┐
                 │  DockerHub  │
                 └──────┬──────┘
                        │
                        ▼
                 ┌─────────────┐
                 │  GitHub     │
                 │ K8s Manifests│
                 └──────┬──────┘
                        │
                        ▼
                 ┌─────────────┐
                 │   Argo CD   │
                 └──────┬──────┘
                        │
                        ▼
                 ┌─────────────┐
                 │ Amazon EKS  │
                 └──────┬──────┘
                        │
             ┌──────────┼──────────┐
             ▼          ▼          ▼
         Frontend    Backend    Database
             │          │
             └────┬─────┘
                  ▼
          Prometheus + Grafana
🎯 Key DevOps Concepts Demonstrated

This project demonstrates practical implementation of:

✅ CI/CD automation
✅ Infrastructure on AWS
✅ Docker containerization
✅ Kubernetes deployment
✅ Amazon EKS
✅ GitOps
✅ Argo CD
✅ Code quality analysis
✅ Dependency vulnerability scanning
✅ Container security scanning
✅ Kubernetes monitoring
✅ Prometheus metrics
✅ Grafana dashboards
✅ Automated Kubernetes manifest updates
✅ Docker image versioning
✅ Automated deployment notifications
💡 Key Learning

The major goal of this project was to understand how individual DevOps tools work together as a complete delivery system.

Code
 ↓
Build
 ↓
Test
 ↓
Scan
 ↓
Containerize
 ↓
Push
 ↓
GitOps
 ↓
Deploy
 ↓
Monitor

This project provided hands-on experience with building and troubleshooting a complete DevOps workflow from source code to production-style Kubernetes deployment.

👨‍💻 Author

Jobin

DevOps / Cloud Engineering Project

⭐ If you found this project useful

Feel free to explore the repository, fork it, and experiment with the CI/CD and GitOps workflow.


### For your GitHub repository

Put the generated architecture image in the repository root as:

```text
architecture-diagram.png

Then this line in the README will display it:

![Wanderlust DevOps Architecture](architecture-diagram.png)

Your repository will then look like:

Wanderlust_3TIER/
├── README.md
├── architecture-diagram.png
├── frontend/
├── backend/
├── kubernetes/
├── Automations/
└── Jenkinsfile

<img width="958" height="503" alt="image" src="https://github.com/user-attachments/assets/49991758-c0a8-40b1-9ff7-43ee7738e2aa" />
<img width="1600" height="890" alt="image" src="https://github.com/user-attachments/assets/6370606c-e41f-412b-b15a-78f0f9ed64db" />
<img width="1564" height="1006" alt="image" src="https://github.com/user-attachments/assets/c222352b-f96d-44ef-9fc4-838b5d3b011f" />
