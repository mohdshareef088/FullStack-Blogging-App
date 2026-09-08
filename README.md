##📘 **FullStack Blogging App – CI/CD**

This is a full‑stack blogging application built with Jenkins, Docker, Terraform, and Kubernetes. It allows users to create, edit, delete, and publish blog posts with authentication and role‑based access. The project is containerized using Docker and deployed using CI/CD pipelines.

---
## **Architecture Diagram**
<img width="1025" height="683" alt="image" src="https://github.com/user-attachments/assets/703a5a70-a5e2-4cf7-8495-8b6596df65b1" />

---
##🏗️ **Architecture Overview**
```
- **Frontend:** GitHub, Jenkins UI, SonarQube dashboard, Grafana, Public access domain URL
- **Backend:** Maven, Trivy, Docker, Kubernetes, Terraform   
- **DevOps:** Docker, Trivy, SonarQube
- **CI/CD:** Jenkins
- **Hosting:** AWS EC2 , AWS EKS, AWS VPC, Subnets, Load Balancer 
```
---
##🛠️ **Prerequisites**
- **Ubuntu 24.04 LTS 
- **Java 21 (OpenJDK)
- **Jenkins - CI/CD  
- **SonarQube - Docker container
- **Nexus-Artifactory - Docker container
- **Monitoring & Observability- Grafana, Prometheus, blackbox
- **Custom ports- 2000-11000 required for Jenkins jobs & monitoring tools

---

## 🚀**Application Deploy**

docker run images 
<img width="1226" height="163" alt="image" src="https://github.com/user-attachments/assets/d90884ac-75ca-4b79-8f0e-f7b9237fa401" />
<img width="997" height="74" alt="image" src="https://github.com/user-attachments/assets/6c7ca432-5710-4e66-a09f-2befe2e74f78" />
---

## Manage Jenkins

- ** Add Java and SonarQube with their credentials to Jenkins tools **
- ** Nexus URL to pom.xml
- ** Pushing image to Docker **  

## Groovy script  

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
eksctl  create cluster --name devopsshack-cluster --region ap-south-1
kubectl create ns  webapps
Svc -- creation to assign a service
Role -- creation to assign to the service account in Jenkins
Bind -- the role to the service account to perform the deployment to update it or delete it
webapps secret -- for pulling the Docker image from my private registry and to be accessed  and utilized by Kubernetes to make pods of this image


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

- Homepage  Dashboard 
<img width="1908" height="970" alt="image" src="https://github.com/user-attachments/assets/e9962fb3-5ba8-474e-b71a-5885b93d2e2b" />

- registration page
<img width="464" height="561" alt="image" src="https://github.com/user-attachments/assets/38d7ac0b-e2d2-42f0-b02e-2f84d38aa568" />

- Create blog page 
<img width="1888" height="787" alt="image" src="https://github.com/user-attachments/assets/8b37a1c2-a22b-4226-bfba-3e749a233d90" />

---

##**Live Demo**
Add link:

```
https://yourapp.com
```


