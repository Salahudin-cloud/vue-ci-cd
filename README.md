
# 📦 Vue 3 + Jenkins + GitHub Webhook + Ngrok (Local Deployment)

This guide helps you set up a **CI/CD pipeline** for a Vue 3 project using **Jenkins**, triggered by **GitHub webhooks**, and served via **ngrok** on a **local server**.

---

## 1️⃣ Prerequisites

- Node.js and npm installed
- Jenkins installed on your local machine (e.g., running at http://localhost:8080)
- A verified **ngrok** account
- A GitHub repository with your Vue project
- Nginx or PM2 (optional) for serving production files

---

## 2️⃣ Install Tools

### Install Jenkins on Ubuntu

```bash
sudo apt update
sudo apt install openjdk-21-jdk -y
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install jenkins -y
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

### Install ngrok

```bash
sudo snap install ngrok
ngrok config add-authtoken <your-authtoken>
```

---

## 3️⃣ Jenkins Setup

### A. Install Required Plugins

- NodeJS Plugin
- Git Plugin
- GitHub Integration Plugin
- Pipeline Plugin

### B. Configure NodeJS Tool

1. Go to **Jenkins → Manage Jenkins → Global Tool Configuration**
2. Scroll to **NodeJS** section
3. Click "Add NodeJS", set name (e.g., `node16`, `node20`, etc.), and check "Install automatically"

### C. Create New Pipeline Job

1. Go to Jenkins → New Item
2. Choose "Pipeline", name it `vue-ci-cd`
3. Under **Pipeline section** → Definition: `Pipeline script from SCM`
4. SCM: Git → Paste your GitHub repo URL
5. Script Path: `Jenkinsfile`

### D. Enable Webhook Trigger

- In the job config, scroll to "Build Triggers"
- ✅ Check "GitHub hook trigger for GITScm polling"

---

## 4️⃣ GitHub Webhook Setup

### A. Start Ngrok

```bash
ngrok http 8080
```

- Copy the `https://xxxx.ngrok.io` URL

### B. Add Webhook to GitHub

1. Go to your GitHub repo → Settings → Webhooks → Add Webhook
2. **Payload URL:** `https://xxxx.ngrok.io/github-webhook/`
3. **Content type:** `application/json`
4. ✅ Just the push event
5. Save

---

## 5️⃣ Jenkinsfile (Place in project root)

```groovy
pipeline {
  agent any

  tools {
    nodejs 'node20'
  }

  environment {
    APP_NAME = "vue-app"
    PORT = "5173"
  }

  stages {
    stage('Clone') {
      steps {
        sh 'git config --global http.postBuffer 524288000'
        dir('vue-ci-cd') {
          retry(2) {
            git url: 'https://github.com/your-user/vue-ci-cd.git', changelog: false, poll: false
          }
        }
      }
    }

    stage('Install Dependencies') {
      steps {
        dir('vue-ci-cd') {
          sh 'npm install -g yarn'
          sh 'yarn install'
        }
      }
    }

    stage('Build') {
      steps {
        dir('vue-ci-cd') {
          sh 'yarn build'
        }
      }
    }

    stage('Deploy with Nginx') {
      steps {
        dir('vue-ci-cd') {
          sh 'sudo rm -rf /var/www/html/*'
          sh 'sudo cp -r dist/* /var/www/html/'
        }
      }
    }

    stage('Serve with PM2 (optional)') {
      steps {
        dir('vue-ci-cd') {
          sh 'npm install -g pm2'
          sh 'pm2 delete ${APP_NAME} || true'
          sh 'pm2 serve dist ${PORT} --name ${APP_NAME} --spa'
          sh 'pm2 save'
        }
      }
    }
  }
}
```

---

## 6️⃣ Test CI/CD

1. Push changes to your GitHub repo (e.g., edit `README.md`)
2. GitHub sends webhook → Jenkins triggers build
3. Jenkins builds → deploys to Nginx or PM2

---

## 7️⃣ Local Network Access

To allow other devices in your LAN:

1. Make sure firewall allows access to port 8080 or 5173
2. Use your local IP (e.g., `http://192.168.1.10`) in browser

---

## ✅ Done

You now have automated deployment for a Vue 3 app from GitHub to your local server using Jenkins and ngrok.
