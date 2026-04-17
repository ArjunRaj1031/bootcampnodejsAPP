# 🚀 DevSecOps Pipeline Setup (Ubuntu 22.04 | AWS & Azure)

---

## 📌 Project Overview

This project demonstrates a complete **DevSecOps CI/CD pipeline**:

```
GitHub → Jenkins → Sonar Scanner → SonarQube → Snyk → Docker → DockerHub
```

---

# 🌐 Step 0: Launch VM (AWS / Azure)

## 🔹 AWS EC2

1. Go to AWS → EC2 → Launch Instance
2. Select:

   * **Ubuntu Server 22.04 LTS**
   * Instance: **t3.medium or higher (8GB recommended)**

### 🔐 Security Group

Add inbound rules:

| Port | Purpose   |
| ---- | --------- |
| 22   | SSH       |
| 8080 | Jenkins   |
| 9000 | SonarQube |

---

## 🔹 Azure VM

1. Go to Azure → Virtual Machines → Create
2. Select:

   * Ubuntu 22.04
   * Size: B2s or higher

### 🔐 Open Ports

Add:

* 22
* 8080
* 9000

---

# 🔐 Step 1: Connect to VM

```bash
ssh -i <key.pem> ubuntu@<PUBLIC-IP>
```

---

# 🔧 Step 2: Install Jenkins (Using Script)

## 📂 Create Script

```bash
vi jenkins_fresh_install.sh
```

👉 Press `i` → paste Jenkins script → `ESC` → `:wq`

---

## ▶️ Run Script

```bash
chmod +x jenkins_fresh_install.sh
sudo ./jenkins_fresh_install.sh
```

---

## 🌐 Access Jenkins

```
http://<PUBLIC-IP>:8080
```

---

## 🔑 Get Password

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

---

# 🔧 Step 3: Install SonarQube (Server)

## 📂 Create Script

```bash
vi sonar.sh
```

---

## ▶️ Run Script

```bash
chmod +x sonar.sh
sudo ./sonar.sh
```

---

## 🌐 Access SonarQube

```
http://<PUBLIC-IP>:9000
```

Login:

```
admin / admin
```

---

# 🧠 SonarQube vs Sonar Scanner (IMPORTANT)

### 🔹 SonarQube

* Web UI (port 9000)
* Stores results
* Dashboard

### 🔹 Sonar Scanner

* CLI tool
* Reads code
* Sends analysis to SonarQube

---

## 🎯 Flow

```
Code → Sonar Scanner → SonarQube → Dashboard
```

👉 Without Scanner → No analysis ❌

---

# 🔧 Step 4: Install Required Tools

```bash
sudo apt update
sudo apt install -y nodejs npm docker.io unzip

sudo systemctl start docker
sudo systemctl enable docker

sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

---

# 🔧 Step 5: Install Sonar Scanner

```bash
cd /opt

sudo wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-6.0.0.4432-linux.zip

sudo unzip sonar-scanner-cli-*.zip

sudo mv sonar-scanner-* sonar-scanner

sudo chmod -R 755 sonar-scanner
```

---

# 🔧 Step 6: Install Snyk

```bash
sudo npm install -g snyk
```

---

# 🔑 Step 7: Generate Tokens

## SonarQube Token

SonarQube → My Account → Security → Generate Token

## Snyk Token

Snyk → Account → API Token

---

# ⚙️ Step 8: Jenkins Setup

1. Open Jenkins
2. Install suggested plugins
3. Create admin user

---

# 📦 Step 9: Create Pipeline

1. Click **New Item**
2. Select **Pipeline**
3. Paste Jenkinsfile

---

# 📜 Jenkinsfile

```groovy
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "your-dockerhub-username/app"
        DOCKER_TAG = "latest"
        SONAR_HOST_URL = "http://<IP>:9000"
        SONAR_TOKEN = "your-sonar-token"
        SNYK_TOKEN = "your-snyk-token"
    }

    stages {

        stage('Check Tools') {
            steps {
                sh 'node -v && npm -v && docker -v'
            }
        }

        stage('Checkout') {
            steps {
                git 'https://github.com/rajcocvs/bootcampnodejsAPP.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Sonar Scan') {
            steps {
                sh '''
                export PATH=$PATH:/opt/sonar-scanner/bin
                sonar-scanner \
                  -Dsonar.projectKey=my-node-app \
                  -Dsonar.sources=. \
                  -Dsonar.host.url=$SONAR_HOST_URL \
                  -Dsonar.login=$SONAR_TOKEN
                '''
            }
        }

        stage('Snyk Scan') {
            steps {
                sh '''
                snyk auth $SNYK_TOKEN
                snyk test || true
                '''
            }
        }

        stage('Docker Build & Push') {
            steps {
                sh '''
                docker build -t $DOCKER_IMAGE:$DOCKER_TAG .
                echo $DOCKER_CREDS_PSW | docker login -u $DOCKER_CREDS_USR --password-stdin
                docker push $DOCKER_IMAGE:$DOCKER_TAG
                '''
            }
        }
    }
}
```

---

# ▶️ Step 10: Run Pipeline

Click:

```
Build Now
```

---

# ✅ Expected Output

✔ SonarQube Analysis Successful
✔ Snyk Scan Completed
✔ Docker Image Pushed

---

# ⚠️ Troubleshooting

## Jenkins not starting

```bash
sudo systemctl status jenkins
```

## SonarQube not starting

```bash
sudo systemctl status sonarqube
```

👉 Wait 1–2 mins for first startup

---

## Permission errors

```bash
sudo ./script.sh
```

---

# 🎯 Final Flow

```
Launch VM → Install Jenkins → Install SonarQube → Install Scanner → Run Pipeline
```

---

# 👨‍💻 Author

Rajco DevSecOps Project
