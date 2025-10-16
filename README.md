# 🧑‍💻 DevSecOps CI/CD Pipeline Project

This project demonstrates a complete **CI/CD pipeline** using **Jenkins**, **SonarQube**, **Docker**, **Trivy**, **Kubernetes**, and **Argo CD** on an **Amazon EC2 (Amazon Linux)** instance.  
It automates code analysis, image scanning, containerization, and deployment using GitOps principles.

---

## 🚀 Project Overview

### Tech Stack
- **Compute**: AWS EC2 (m7i-flex large, Amazon Linux)
- **CI/CD**: Jenkins
- **Code Quality**: SonarQube
- **Containerization**: Docker
- **Security Scanning**: Trivy
- **Orchestration**: Kubernetes
- **GitOps Deployment**: Argo CD

---

## 🧩 Setup Steps

### Step 1: EC2 Instance Setup
- Launched an **m7i-flex large** EC2 instance with **Amazon Linux**.
- Installed required packages:
  ```bash
  sudo yum update -y
  sudo yum install -y docker git
  sudo systemctl start docker
  sudo systemctl enable docker
  ```

---

### Step 2: Install Jenkins and SonarQube
- Installed Jenkins and started the service.
- Installed required Jenkins plugins:
  - **Pipeline**
  - **Git**
  - **SonarQube Scanner**
  - **Docker Pipeline**
  - **Email Extension**
- Created a **SonarQube container** using Docker:
  ```bash
  docker run -d --name sonarqube -p 9000:9000 sonarqube:lts-community
  ```

---

### Step 3: Jenkins Pipeline Configuration

Below is the Jenkins Declarative Pipeline used for automating the CI/CD process.

```groovy
pipeline {
    agent any
    environment {
        SCANNER_HOME = tool 'mysonar'
    }
    stages {
        stage('CleanWS') {
            steps {
                cleanWs()
            }
        }
        stage('Code') {
            steps {
                git "https://github.com/abdulshaik0811/ltibbhackathon.git"
            }
        }
        stage('Sonarqube Analysis') {
            steps {
                withSonarQubeEnv('mysonar') {
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner                     -Dsonar.projectName=zomato                     -Dsonar.projectKey=zomato
                    '''
                }
            }
        }
        stage('Quality Gates') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonarcredentials'
                }
            }
        }
        stage('Image Build') {
            steps {
                sh 'docker build -t appimage .'
                sh 'docker build -t dbimage database/'
            }
        }
        stage('Image Scan') {
            steps {
                sh 'trivy image appimage'
                sh 'trivy image dbimage'
            }
        }
        stage('Tag & Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub') {
                        sh 'docker tag dbimage abdulshaik0811/clientapp:dbimage'
                        sh 'docker tag appimage abdulshaik0811/clientapp:appimage'
                        sh 'docker push abdulshaik0811/clientapp:dbimage'
                        sh 'docker push abdulshaik0811/clientapp:appimage'
                    }
                }
            }
        }
    }
    post {
        always {
            mail to: "shaikabduul36@gmail.com",
            subject: "Pipeline Status",
            body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} \nBuild ${env.BUILD_NUMBER} \nMore info at: ${env.BUILD_URL}"
        }
    }
}
```

---

### Step 4: Kubernetes & Argo CD Deployment

- Created the following Kubernetes manifests:
  - `deployment.yaml`
  - `service.yaml`
  - `hpa.yaml`

- Deployed these using **Argo CD**:
  1. Created an Argo CD **Project**.
  2. Linked the **GitHub repo** containing manifests.
  3. Argo CD continuously syncs changes from Git → Kubernetes (GitOps model).

---

## ⚙️ Pipeline Flow

1. **Clean Workspace**  
   Clears old workspace data.

2. **Checkout Code**  
   Pulls source code from GitHub.

3. **Code Quality Check**  
   Runs SonarQube analysis for bugs, vulnerabilities, and code smells.

4. **Quality Gate**  
   Validates SonarQube results before proceeding.

5. **Build Docker Images**  
   Builds application (`appimage`) and database (`dbimage`) Docker images.

6. **Security Scan**  
   Scans images using **Trivy** for vulnerabilities.

7. **Push to DockerHub**  
   Tags and pushes images to DockerHub repository:
   - [`abdulshaik0811/clientapp:appimage`](https://hub.docker.com/r/abdulshaik0811/clientapp)
   - [`abdulshaik0811/clientapp:dbimage`](https://hub.docker.com/r/abdulshaik0811/clientapp)

8. **Deployment via Argo CD**  
   Argo CD automatically deploys images to Kubernetes based on updated manifests.

---

## 📊 Tools & Integrations Summary

| Tool | Purpose |
|------|----------|
| **Jenkins** | Automates build and deployment pipeline |
| **SonarQube** | Code quality and vulnerability analysis |
| **Docker** | Containerization of application and database |
| **Trivy** | Container image vulnerability scanning |
| **Kubernetes** | Orchestrates and manages containerized workloads |
| **Argo CD** | Automates deployment using GitOps |

---

## 📬 Notifications
- Jenkins sends an email notification to **shaikabduul36@gmail.com** after every pipeline run with build results and a direct job link.

---

## 📁 Repository Structure

```
.
├── Dockerfile
├── database/
│   └── Dockerfile
├── deployment.yaml
├── service.yaml
├── hpa.yaml
└── Jenkinsfile
```

---

## 🧠 Future Enhancements
- Integrate **Prometheus** and **Grafana** for monitoring.
- Add **Helm** charts for Kubernetes deployments.
- Enable **Slack notifications** for build updates.

---

## 👨‍💻 Author
**Abdul Shaik**  
📧 [shaikabduul36@gmail.com](mailto:shaikabduul36@gmail.com)  
🔗 [GitHub: abdulshaik0811](https://github.com/abdulshaik0811)

---

⭐ *If you found this project helpful, consider giving it a star on GitHub!*
