FullStack Blogging App – MERN + Docker + CI/CD

 Project Overview

This is a full‑stack blogging application built using React, Node.js, Express, and MongoDB. It allows users to create, edit, delete, and publish blog posts with authentication and role‑based access. The project is containerized using Docker and deployed using CI/CD pipelines.

 Tech Stack
List everything clearly:

Frontend: React, TailwindCSS

Backend: Node.js, Express.js

Database: MongoDB

Authentication: JWT

DevOps: Docker, Docker Compose

CI/CD: GitHub Actions

Hosting: AWS EC2 / Render / Vercel (whatever you used)

Architecture Diagram
<img width="1025" height="683" alt="image" src="https://github.com/user-attachments/assets/41382b86-be1c-4347-89fd-549f5f61773b" />

Project Structure
Code
/client      → React frontend
/server      → Node.js backend
/docker      → Dockerfiles & compose
/.github     → CI/CD workflows
Repo-Git
Required Tools-
Jenkins- Build and push code

Trivy -Scan bugs
Sonarqube- to scan the code
Maven -dependencies to build jarfile
Nevus Arti- to push the code
Docker -to create docker image
Trivy -Scan image

Required servers-
Jenkins - ubuntu 24.04 LTS 
Sonarqube
Nexus-Artifactory
Monitoring Tolls
Custom ports- 2000-11000


sudo apt install openjdk-17-jre-headless 

dpkg -l | grep openjdk-17 - uninstall package

JAVA- ubuntu 24.04 and jenkins now runs on java 21 with all plugins downloads 
sudo apt update
sudo apt install openjdk-21-jdk -y
sudo update-alternatives --config java
sudo update-alternatives --config javac
java -version

JENKINS
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install jenkins
05d89a8620f742f18c638ece0ec38af7

Interview question
Jenkins Plugins: 1. SonarQube Scanner 2. Config File Provider 3. Maven Integration 4. Pipeline Maven Integration 5. Kubernetes Credentials 6. Kubernetes 7. Kubernetes CLI 8. Kubernetes Client API 9. Docker Pipeline 10.docker

From <https://www.youtube.com/watch?v=kWON8yc6efU> 


Docker
Sudo apt install docker.io -y
Sudo usermod -aG docker ubuntu 
Sudo usermod -aG docker jenkins 
sudo usermod -aG docker $USER
sudo usermod -aG docker sonarqube
newgrp docker
sudo chmod 666 /var/run/docker.sock

sudo journalctl -xeu jenkins -logs



Nexus image on nexus server
<img width="1226" height="163" alt="image" src="https://github.com/user-attachments/assets/2d79b476-2bb9-4d65-a5c8-5376f143df67" />


sudo docker run -d -p 8081:8081 sonatype/nexus3
<img width="875" height="89" alt="image" src="https://github.com/user-attachments/assets/26154796-a01a-4b40-a7c5-a2889e3ab21c" />

sudo docker exec -it 92d961c37040 /bin/sh

Password b0c53eee-aefa-4d7d-9675-622aecd59eaf
<img width="997" height="74" alt="image" src="https://github.com/user-attachments/assets/b76166f0-a2a1-403b-9fe9-009b1a38750f" />


sudo docker run -d -p 9000:9000 sonarqube:lts-community


Sonar token
squ_fe39d088827d3abea47d6255a6ee06ea4eab37a3

Add in jenkins credentials keys

Git token
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


Plugin config system with servers urls and tools with plugin install sonar,maven,docker,git

<img width="869" height="208" alt="image" src="https://github.com/user-attachments/assets/7ed56acf-e908-4873-adf0-4b577e7914d1" />


Configure plugin in jenkins tools
Docker,maven,sonar-scanner

To configure install another version of java

<img width="1576" height="630" alt="image" src="https://github.com/user-attachments/assets/78a8166a-3fef-44f3-8fcd-0fac06d2bd04" />


Add pipeline stage view to know the stages 

Pipeline code


Adding stages

Write git stage in pipeline syntax with include credentials
	1. With the help of pipeline syntax
<img width="936" height="830" alt="image" src="https://github.com/user-attachments/assets/6ccc3217-7e97-4ae4-887d-949f135f55de" />



Sonarqube server - installed on server
Sonarqube scanner - installed on jenkins


3.Sonarqube scanner - installed on jenkins

<img width="1320" height="580" alt="image" src="https://github.com/user-attachments/assets/b197a074-6208-4b91-8d73-b3365bf4d966" />

4.maven-settings to communicate with nexus
Change pom.xml maven-releases url 

<img width="1241" height="611" alt="image" src="https://github.com/user-attachments/assets/8f208dfa-7acc-4ceb-af5d-89c38678047f" />

<img width="995" height="507" alt="image" src="https://github.com/user-attachments/assets/0a43f426-a488-4c61-9f7a-e843a0d23119" />

<img width="1174" height="903" alt="image" src="https://github.com/user-attachments/assets/75bdaa39-cb4d-4eb6-9202-91d83288d09e" />



	5. Check jdk 21 installed on server
  <img width="671" height="85" alt="image" src="https://github.com/user-attachments/assets/246aad8b-3dbb-4b33-a29b-df4502d45b1d" />

	6. 
	


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
    


Dsonar are parameters
Thing	Why it exists	What happens if missing
url: https://index.docker.io/v1/	Jenkins needs registry login endpoint	Push fails with 401
--platform linux/amd64	Buildx must know architecture	Image not loaded, Trivy fails, push fails
error

Error with docker user permission usermod jenkins fixed
Sonarqube crashed once,


From <edge://discover-chat-v2/> 

Trivy stage
It is a tools used to scan dependencies scan and file scan

<img width="1867" height="842" alt="image" src="https://github.com/user-attachments/assets/90d842bc-9ced-45ba-9977-5c187cf558f6" />


Problem: pom.xml and maven push code deploy settings.xml should be same
<img width="616" height="285" alt="image" src="https://github.com/user-attachments/assets/6f573c4f-c851-410d-9f35-f3fd508f3641" />

<img width="537" height="763" alt="image" src="https://github.com/user-attachments/assets/ee857a23-f9cf-4161-a737-084b1de17856" />

<img width="1580" height="734" alt="image" src="https://github.com/user-attachments/assets/f7fab4fe-8620-4ec2-96e4-2b0c9e14b83b" />

<img width="1355" height="495" alt="image" src="https://github.com/user-attachments/assets/c75c8170-69db-4621-af64-141c220dd8a9" />






Docker pipeline syntax
<img width="674" height="800" alt="image" src="https://github.com/user-attachments/assets/4ce32eda-613a-4b27-aa76-7bea60e5c9e5" />


<img width="1879" height="671" alt="image" src="https://github.com/user-attachments/assets/5cb95ca8-25f1-456a-a214-50967fdbfb9a" />


<img width="1859" height="636" alt="image" src="https://github.com/user-attachments/assets/e3a52770-4e85-4dc5-a946-4da649f1ac49" />




sudo apt update && sudo apt install curl unzip -y
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
awscliv2.zip
sudo ./aws/install
sudo snap install terraform --classic


AKIASBM2MX6I27S3UOAU
Bs4sxK8hSksIpaVi8q58wRvEyZhKToIMH+WIieAv

Installing kubectl to access nodes
sudo apt install kubectl --classic


eksctl  create cluster --name testing-cluster --region ap-south-1 -optional
eksctl delete cluster --region=ap-south-1 --name=testing-cluster



Create cluster using terraform and update config file 



aws eks update-kubeconfig --region ap-south-1 --name devopsshack-cluster -- config file to connect to nodes
kubectl get nodes

<img width="886" height="137" alt="image" src="https://github.com/user-attachments/assets/ff647637-6f94-45f5-9b3e-f128a0a1ba94" />


To give access to kubernetes we need RBAC

First create namespace 

kubectl create ns  webapps
namespace/webapps created

Svc account for jenkins 
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins
  namespace: webapps

Role back access control

kubectl apply -f svc.yml
serviceaccount/jenkins created


 kubectl apply -f role.yml
role.rbac.authorization.k8s.io/app-role created

Role -- creation to assign to service account in jenkins

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: full-access-role
rules:
  - apiGroups:
      - ""
      - apps
      - autoscaling
      - batch
      - extensions
      - policy
      - rbac.authorization.k8s.io
    resources:
      - pods
      - secrets
      - componentstatuses
      - configmaps
      - daemonsets
      - deployments
      - events
      - endpoints
      - horizontalpodautoscalers
      - ingresses
      - jobs
      - limitranges
      - namespaces
      - nodes
      - persistentvolumes
      - persistentvolumeclaims
      - resourcequotas
      - replicationcontrollers
      - replicasets
      - serviceaccounts
      - services
    verbs:
      - get
      - list
      - watch
      - create
      - update
      - patch
      - delete

Bind -- the role to the service account to perform the deployment to update it or delete it
 kubectl apply -f bind.yml
rolebinding.rbac.authorization.k8s.io/app-rolebinding created

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-rolebinding
  namespace: webapps
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: app-role
subjects:
  - kind: ServiceAccount
    name: jenkins
    namespace: webapps

Now we  get access to perform the deployment update and delete

Token for service account for jenkins

kubectl apply -f jenkins-secret.yml -n webapps
secret/mysecretname created

apiVersion: v1
kind: Secret
metadata:
  name: mysecretname
  namespace: webapps
  annotations:
    kubernetes.io/service-account.name: jenkins
type: kubernetes.io/service-account-token

webapps secret for pulling the docker image from my private registry and to be accessed  to be utilized for Kubernetes to make pods of this image
kubectl create secret docker-registry regcred \
  --docker-server=https://index.docker.io/v1/ \
  --docker-username=mohdshareef088 \
  --docker-password=F@hd**61914 \
  --namespace=webapps

Create Secret file to be used for authentication from jenkins to kubenetes 

kubectl get secrets -n webapps
kubectl describe secret mysecretname -n webapps
<img width="1656" height="433" alt="image" src="https://github.com/user-attachments/assets/9ac2ae2f-2893-4e80-83bc-ee7c0c86a546" />


Add to secret text
eyJhbGciOiJSUzI1NiIsImtpZCI6ImgxMWVsUHkybDlyalBYR1FxYjQ2R1ZCc19MUzZvVUg5ZW5QcFBMT0dkaVEifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJ3ZWJhcHBzIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6Im15c2VjcmV0bmFtZSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJqZW5raW5zIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiZTUxZjdmZjgtOTY0Yy00NTU4LWEwYTYtNjYxYjJiZTNjZWFmIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50OndlYmFwcHM6amVua2lucyJ9.LUGxxMBSED554pGLG8_JIrY75-un1qIF8i62zdxHAVdzXmmvTQF4HaRpCFCKq4eEJGDuzvRD3BmOridJI-TepCWGMcvUuS7CUqIt0iPg64_R4ZIcNU7XkwJ0ibQxHTqX486ZTmwHR-ZGssF3RYoRguyXcppimD6eDi5opBQSsDrJpyyE3DE-KJqdrLTsVqSyr3p1-G5rk7OuB7oEmN1uWoLl-bL92ubP-Itv6gmhs0tq-AT6klibLrLVW-shN-g1mZcCr5enwwNbk1527LVsPB2vRAukOippcykQWNcNuTP3CZL0eV92x-DGgd-5SWuAqX24LGo4uIhQ3HKUltPeVA

<img width="716" height="422" alt="image" src="https://github.com/user-attachments/assets/758593bd-433a-4d22-938d-0cbe4072689a" />

<img width="1781" height="896" alt="image" src="https://github.com/user-attachments/assets/12d54103-da5b-4cea-9659-23921df95daa" />

<img width="1897" height="725" alt="image" src="https://github.com/user-attachments/assets/163b7d40-5dea-4c4e-88da-f00965c65a8e" />

<img width="1881" height="802" alt="image" src="https://github.com/user-attachments/assets/f2c8b41f-6c47-4d3a-a12a-19d8cc2af31b" />



To authenticate roles is working should return yes

kubectl auth can-i create deployments --as=system:serviceaccount:webapps:jenkins -n webapps
kubectl auth can-i create services --as=system:serviceaccount:webapps:jenkins -n webapps


<img width="950" height="180" alt="image" src="https://github.com/user-attachments/assets/c5ebdc89-a196-43ff-931b-110d046d78db" />

<img width="1171" height="163" alt="image" src="https://github.com/user-attachments/assets/18d297d2-2c72-46c7-9c79-37f6f2892945" />






Autoscaling k8 nodes
 aws eks update-nodegroup-config   --cluster-name devopsshack-cluster   --nodegroup-name devopsshack-node-group   --scaling-config minSize=2,maxSize=2,desiredSize=2   --region ap-south-1

<img width="1908" height="970" alt="image" src="https://github.com/user-attachments/assets/1b8a5044-7f23-45ad-8bed-3007a4696842" />

<img width="464" height="561" alt="image" src="https://github.com/user-attachments/assets/e79d6ef4-986f-44eb-abbf-fcda8e8b6340" />

<img width="1888" height="787" alt="image" src="https://github.com/user-attachments/assets/5564e277-df74-44fd-b8cf-08798aef0eef" />









DELETE THE LOAD BALANCER AND SECURITY GRP to delete the vpc id there is an error

Monitor 

docker run -d -p 3000:3000 grafana/grafana
docker run -d -p 9115:9115 prom/blackbox-exporter


Extract 
wget https://github.com/prometheus/prometheus/releases/download/v3.13.2/prometheus-3.13.2.linux-amd64.tar.gz

https://github.com/prometheus/blackbox_exporter/releases/download/v0.28.0/blackbox_exporter-0.28.0.linux-amd64.tar.gz

sudo apt-get install -y adduser libfontconfig1 musl
wget https://dl.grafana.com/grafana-enterprise/release/13.2.0/grafana-enterprise_13.2.0_32077357341_linux_amd64.deb
sudo dpkg -i grafana-enterprise_13.2.0_32077357341_linux_amd64.deb










prometheus/blackbox_exporter: Blackbox prober exporter  add the prometurs config inside prometues yml file Modily the yml file inside prometues folder 



nano prometheus.yml


- job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]
       # The label name is added as a label `label_name=<label_value>` to any timeseries scraped from this config.
        labels:
          app: "prometheus"


            - job_name: 'blackbox'
    metrics_path: /probe
    params:
      module: [http_2xx]  # Look for a HTTP 200 response.
    static_configs:
      - targets:
        - http://prometheus.io    # Target to probe with http.
        - https://prometheus.io   # Target to probe with https.
        - http://a8220d79f98dc4a62b755e6eaa212597-2071549167.ap-south-1.elb.amazonaws.com/login # Target to probe with http on port 8080.
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 127.0.0.1:9115  # The blackbox exporter's real hostname:port.
  - job_name: 'blackbox_exporter'  # collect blackbox exporter's operational metrics.
    static_configs:
      - targets: ['127.0.0.1:9115']
~
Kill and restart pro after entering application url as shown above



<img width="1888" height="938" alt="image" src="https://github.com/user-attachments/assets/38ba155f-5933-4b4f-b513-38f9970505f5" />





7587 - blackbox id 



Feed for monitoring url


<img width="1888" height="938" alt="image" src="https://github.com/user-attachments/assets/e3c32e2e-d5ad-4f0e-a5e0-03815a6d83cd" />

<img width="1842" height="645" alt="image" src="https://github.com/user-attachments/assets/39936e97-25ec-4f68-b4c3-78a10aa94512" />

global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'blackbox'
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
        - https://google.com
        - https://github.com
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: blackbox:9115

docker stop prometheus
docker rm prometheus

docker run -d -p 9090:9090 prom/prometheus

docker run -d \
  -p 9090:9090 \
  -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml \
  --name prometheus \
  prom/prometheus


Mohammed — this screenshot tells me EXACTLY what the problem is now, and the fix is very simple.
Your Prometheus is trying to reach Blackbox Exporter at:
http://blackbox:9115
But Docker cannot resolve the hostname blackbox, so Prometheus shows:
lookup blackbox: no such host
This means:
⭐ Prometheus and Blackbox Exporter are NOT on the same Docker network.
Because of that, Prometheus cannot reach Blackbox using the service name.
Let’s fix it cleanly.

✅ THE FIX: Put both containers on the same Docker network
Step 1 — Create a Docker network
docker network create monitoring
Step 2 — Remove old containers
docker rm -f prometheus
docker rm -f laughing_mendel
Step 3 — Start Blackbox Exporter on that network
docker run -d \
  --name blackbox \
  --network monitoring \
  -p 9115:9115 \
  prom/blackbox-exporter
Step 4 — Start Prometheus on the same network
Make sure you are inside:
~/prometheus
Then run:
docker run -d \
  --name prometheus \
  --network monitoring \
  -p 9090:9090 \
  -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus

⭐ Why this works
Inside a Docker network:
	• Container names become DNS hostnames
	• Prometheus can now reach Blackbox at:
http://blackbox:9115
Your error will disappear.

⭐ After restarting, check Prometheus targets
Open:
http://<server-ip>:9090/targets
You should now see:
	• blackbox job → UP
	• prometheus job → UP

⭐ If you want, I can give you a full docker‑compose.yml
This will run:
	• Prometheus
	• Grafana
	• Blackbox Exporter
All with one command:
docker compose up -d
Just say “give me docker compose”.

From <edge://discover-chat-v2/> 

Add prometeus url in garafana


<img width="1107" height="768" alt="image" src="https://github.com/user-attachments/assets/06212a3b-262f-48df-8d68-aca12e3d3725" />





7587 -blackbox id 


<img width="1152" height="767" alt="image" src="https://github.com/user-attachments/assets/b3789ccf-d8fa-42d4-a74b-498872e98ecd" />


<img width="1164" height="665" alt="image" src="https://github.com/user-attachments/assets/18fbf461-a2aa-453f-b01a-073b3aeabafa" />


