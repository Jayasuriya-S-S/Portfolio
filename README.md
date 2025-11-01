# Portfolio
# DevOps Capstone Project - Portfolio CI/CD Pipeline

##  Overview
This project demonstrates a **complete DevOps CI/CD pipeline** for deploying a **Node.js portfolio application** using **Jenkins, Docker, GitHub, and AWS EC2**, with monitoring via **Prometheus, Grafana, Node Exporter, and cAdvisor**.

---

##  Project Workflow

### 1. Source Code Management (GitHub)
- The Node.js portfolio web app is hosted on GitHub.
- Webhook is configured to automatically trigger Jenkins on every code push.

### 2. Continuous Integration (Jenkins)
- Jenkins fetches the latest code from GitHub.
- Installs dependencies using `npm install`.
- Builds and tags a Docker image for the app.
- Pushes the image to **DockerHub** automatically.

### 3. Continuous Deployment (EC2)
- Jenkins SSHs into an AWS EC2 instance.
- Pulls the latest Docker image from DockerHub.
- Stops and removes the old container if running.
- Deploys the latest version using:
  ```bash
  docker run -d -p 80:3000 --name portfolio <dockerhub-username>/portfolio-app
  ```

### 4. Monitoring Setup
- **Prometheus** monitors application metrics.
- **Node Exporter** collects system metrics (CPU, memory, disk).
- **cAdvisor** monitors Docker containers.
- **Grafana** visualizes data and sends alerts (configured via email/webhook).

### 5. Log Backup Automation
- A **cron job** runs daily at midnight to back up app logs and metrics:
  ```bash
  0 0 * * * /home/ubuntu/backup_logs.sh
  ```

---

##  Tools & Technologies

Tool  Purpose 

Jenkins - Automates CI/CD pipeline 
Docker - Containerization of the app 
GitHub - Source code version control 
AWS EC2 - Application hosting 
Prometheus - Metrics collection 
Grafana - Visualization and alerting 
Node Exporter - EC2 instance monitoring 
cAdvisor - Docker container monitoring 
Cron Job - Automated log backups 

---



##  Jenkins Pipeline (Jenkinsfile)

```groovy
pipeline {
    agent any

    environment {
        IMAGE = "jaiswathi1234/portfolio-app"
        CRED = "Dcokerhub"
        EC2_USER = "ubuntu"
        EC2_IP = "13.200.210.146"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Jayasuriya-S-S/Portfolio.git'
            }
        }

        stage('Build Node Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Docker Build & Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: CRED, usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                    docker build -t $IMAGE .
                    echo "$PASS" | docker login -u "$USER" --password-stdin
                    docker push $IMAGE
                    '''
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ec2-key', keyFileVariable: 'KEY_FILE', usernameVariable: 'SSH_USER')]) {
                    sh '''
                    ssh -i $KEY_FILE -o StrictHostKeyChecking=no $SSH_USER@$EC2_IP "
                        docker pull $IMAGE &&
                        docker stop portfolio || true &&
                        docker rm portfolio || true &&
                        docker run -d -p 80:3000 --name portfolio $IMAGE
                    "
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
```

---

## Monitoring Configuration

### Prometheus Targets (prometheus.yml)
yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "node_exporter"
    static_configs:
      - targets: ["172.31.30.24:9100"]

  - job_name: "cadvisor"
    static_configs:
      - targets: ["172.31.30.24:8080"]
```

---

## Alerts (Grafana)
- Alert triggers when **CPU usage > 80%**
- Formula:
  ```
  100 - (avg by(instance)(rate(node_cpu_seconds_total{mode="idle"}[2m])) * 100)
  ```
- Notifications: Email or webhook to admin.

---



---

##  Results
- Fully automated CI/CD pipeline.
- Application deployment without manual steps.
- Real-time monitoring with Prometheus & Grafana.
- Email/webhook alerts for performance issues.
- Automated log backups via cron job.

---

##  Key Learnings
- Complete DevOps cycle from source → deploy → monitor.
- Real-time alerting and visualization setup.
- AWS EC2 networking and security group management.
- Understanding CI/CD best practices and troubleshooting.
- Confidence in end-to-end DevOps project delivery.

---

##  Author
Jaya Suriya S S 
DevOps Engineer | AWS | Jenkins | Docker | Prometheus | Grafana  
GitHub: [Jayasuriya-S-S](https://github.com/Jayasuriya-S-S)

---
