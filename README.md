##📘 **FullStack Blogging App – CI/CD**

This is a full‑stack Java-based blogging application built with Jenkins, Docker, Terraform, and Kubernetes is a DevOps automation workflow. It allows users to create, edit, delete, and publish blog posts with authentication and role‑based access. The project is containerized using Docker automated with kubernetes and deployed using jenkins CI/CD pipelines.

---
## **Architecture Diagram**
<img width="1025" height="683" alt="image" src="https://github.com/user-attachments/assets/703a5a70-a5e2-4cf7-8495-8b6596df65b1" />

---
##🏗️ **Architecture Overview**
```
bash-scripts
│install_jenkins.sh
│install_docker.sh
│install_blackbox.sh
│prometheus.yml
│grafana_dashboard.json
kubernetes
│deployment.yml
│service.yml
│role.yaml
│rolebinding.yaml
│serviceaccount.yaml
terraform
│main.tf
│variables.tf
│outputs.tf
src
│app.js
│Dockerfile
README.md
```
---
##🛠️ **Prerequisites and tools used**
- **Ubuntu 24.04 LTS 
- **Java 21 (OpenJDK)
- **Jenkins - CI/CD Continuous Integration and Continuous Deployment)
- **SonarQube - Docker container (Static code analysis for quality and security checks)
- **Nexus-Artifactory - Docker container (Artifact repository manager)
- **Docker: Containerization of the application.
- **Kubernetes (EKS): Orchestration of containerized applications.
- **Monitoring & Observability- Grafana, Prometheus (application performance), Blackbox for availability and uptime.
- **Custom ports- 2000-11000 required for Jenkins jobs & monitoring tools
---

## Executions

1. **GitHub Integration**:
    - Jenkins pulls the latest changes from the GitHub repository and triggers
2. **Jenkins Manage**:
    - Adding tools token,urls,credentials(git,sonar,maven,docker) to Jenkins Manage (systems/tools) to trigger continuous integration 
4. **Maven Build & Test**:
   - The code is compiled and built using Maven into a .jar file
5. **Code Quality Scan**:
   - Code analysis is done using SonarQube.
   - Trivy scans for vulnerabilities.
6. **Docker & Nexus**:
   - Jenkins builds a Docker image for the application.
   - The image is tagged and pushed to DockerHub.
   - Artifacts are pushed to Nexus.
7. **Kubernetes Deployment**:
   - The app is deployed to an EKS cluster.
   - Kubernetes handles scaling and service management.
8. **Monitoring**:
   - Prometheus scrapes metrics from the Blackbox Exporter.
   - Grafana visualizes application uptime, health, and other critical metrics.

---
## 🚀**Application Build & Deploy to Dockerhub**

    pipeline {
    agent any
	
    tools {
        jdk 'jdk21'
        maven 'maven3'
    }

    environment {
        SCANNER_HOME = tool 'sonar-server'
    }

    stages {

        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/mohdshareef088/FullStack-Blogging-App.git'
            }
        }
   
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }

        stage('Test') {
            steps {
                sh "mvn test"
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs --format table -o fs.html ."
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=Blogging-app \
                        -Dsonar.projectKey=Blogging-app \
                        -Dsonar.java.binaries=target
                    '''
                }
            }
        }
        stage('Publish Artifacts') {
            steps {
                withMaven(globalMavenSettingsConfig: 'maven-settings',jdk: 'jdk21', maven: 'maven3',traceability: true) {
                    sh "mvn deploy"
                }
            }
        }
       stage('docker build') {
    steps {
        script {
            withDockerRegistry(credentialsId: 'docker-cred', url: 'https://index.docker.io/v1/') {
                sh "docker build -t mohdshareef088/blogging-app:latest ."
            }
         }
       }
     }
     stage('Trivy image Scan') {
    steps {
        sh "trivy image --format table -o image.html mohdshareef088/blogging-app:latest"
      }
    }
    stage('docker push') {
    steps {
        script {
            withDockerRegistry(credentialsId: 'docker-cred', url: 'https://index.docker.io/v1/') {
                sh "docker push mohdshareef088/blogging-app:latest"
            }
           }
         }
       }
      }
     }

<img width="1580" height="734" alt="image" src="https://github.com/user-attachments/assets/2c133e4f-dd7a-490a-a72d-4807f03a0631" />
<img width="1355" height="495" alt="image" src="https://github.com/user-attachments/assets/d0ad76ca-b533-4fa7-8fd4-db86ae2bf99e" />
<img width="1859" height="636" alt="image" src="https://github.com/user-attachments/assets/70c24df2-47e2-4270-a0ba-50d2709d21f5" />

---
## **kubernetes deploy**
## Install
- **eksctl -- create eks cluster and update the config file 
- **kubectl -- create namespace webapps 
- **Svc -- create to assign a service nodeport
- **Role -- creation to assign to the service account in Jenkins
- **Bind -- the role to the service account to perform the deployment to update it or delete it
- **webapps secret -- for pulling the Docker image from my private registry and to be accessed  and utilized by Kubernetes to make pods of this image
- **pods -- to get the load balancer url to access the application


<img width="1656" height="433" alt="image" src="https://github.com/user-attachments/assets/c5f2dc5e-5ab7-40f8-ae12-be647f96b98b" />

<img width="886" height="137" alt="image" src="https://github.com/user-attachments/assets/eab0f02b-58e7-44ee-add5-0895c5280b1c" />

Add to secret text
<img width="716" height="422" alt="image" src="https://github.com/user-attachments/assets/9accf298-48fb-474b-8efe-af69d3483bae" />

<img width="1897" height="725" alt="image" src="https://github.com/user-attachments/assets/8bc0c261-d060-4cfb-bc9f-b79c70f8af6b" />

<img width="1881" height="802" alt="image" src="https://github.com/user-attachments/assets/e6e8c3a4-a9f2-45d3-baaf-5bf5c51975fa" />
To authenticate roles is working should return yes
<img width="950" height="180" alt="image" src="https://github.com/user-attachments/assets/2f5de54d-f04f-4493-b974-5d8ff44fb87c" />

<img width="1171" height="163" alt="image" src="https://github.com/user-attachments/assets/1b1a9478-b34a-46dc-9440-24ee1a591687" />

---

## **Screenshots**
---
- Homepage  Dashboard 
<img width="1908" height="970" alt="image" src="https://github.com/user-attachments/assets/e9962fb3-5ba8-474e-b71a-5885b93d2e2b" />

- registration page
<img width="464" height="561" alt="image" src="https://github.com/user-attachments/assets/38d7ac0b-e2d2-42f0-b02e-2f84d38aa568" />

- Create blog page 
<img width="1888" height="787" alt="image" src="https://github.com/user-attachments/assets/8b37a1c2-a22b-4226-bfba-3e749a233d90" />
---

## **Observibility**
## Install
- **install -- prometheus,blackbox_exporter, Grafana accesss throught separate ports 
- **configure  -- add connection url to garafana datasources & import balckbox id to grafana dashboard
---


## **Screenshots**
---

<img width="1858" height="589" alt="image" src="https://github.com/user-attachments/assets/908607ee-a526-4b71-81f1-61c423ce0b73" />
<img width="1888" height="938" alt="image" src="https://github.com/user-attachments/assets/acfaaff2-8e71-4a90-ae80-6eb6c744dcac" />
<img width="1107" height="768" alt="image" src="https://github.com/user-attachments/assets/1370dc31-861a-4c03-bb32-07ed25001696" />
<img width="1888" height="938" alt="image" src="https://github.com/user-attachments/assets/bb1788ab-351e-4f89-aae2-cee3737f4937" />




