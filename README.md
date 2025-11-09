# 🚀 Student App — CI/CD with Jenkins, GitHub & Maven 

>This project demonstrates a **complete CI/CD pipeline** for a **Java (Maven)** application using **Jenkins and GitHub Webhooks**, fully automated from **code push → build → test → deploy**.<br>
>This documentation is beginner-friendly as well as professional.

---
<p align="center">
  <img src="https://github.com/nikiimisal/Project-Student-App-using-java-CICD-with-Jenkins-GitHub-Maven/blob/main/img/ci-cd-java-banner.png?raw=true" width="700" alt="Java CI/CD Pipeline Overview">
</p>

## 🧩 Overview

---

* Automates the deployment of a **Java Maven application** using Jenkins CI/CD.  
* Integrates **GitHub Webhooks** to automatically trigger builds when new code is pushed.  
* Pulls the latest source code from the GitHub repository.  
* Builds and tests the Java project using **Maven**.  
* Packages the application into a `.jar` or `.war` file.  
* Deploys the application automatically to the target environment (Tomcat / Local).  
* Ensures **Continuous Integration** and **Continuous Delivery (CI/CD)** for reliable and fast deployments.  
* Provides a fully automated workflow — from code commit to deployment.

---

## 🧰 Tools & Technologies

* **Jenkins** – CI/CD automation  
* **Java (JDK 17)** – Application runtime  
* **Maven** – Build automation tool  
* **GitHub Webhooks** – Trigger builds on code push  
* **EC2 (Ubuntu) (Tomcat)**– Target deployment environment
---
## 🔌Plugins (at minimum)

* **Git Plugin** → For repo clone/pull mechanism  
* **GitHub Plugin** → Auto-trigger builds via Webhook (on push)  
* **Pipeline Plugin** → Defines CI/CD pipeline stages  
* **SSH Agent Plugin** → Enables remote server access (for deployment)  
* **Pipeline Stage View Plugin** → Visual stage-wise pipeline status  

---
## 📋 Prerequisites

* GitHub repository for your Student App  
* Jenkins server (Ubuntu or Windows) reachable by GitHub  
* JDK 17 & Maven installed and configured in Jenkins  
* Target server (optional) with Tomcat for deployment
* java application source code hosted on a Git repository (e.g., GitHub).

---

## High-level CI/CD Flow

1. Developer pushes code to the `main` branch  
2. GitHub Webhook triggers Jenkins build  
3. Jenkins pulls the latest code from GitHub  
4. Jenkins builds and tests the app using Maven  
5. Jenkins packages the `.jar` / `.war` file  
6. Jenkins deploys the app to the target server (Tomcat or Local)  
7. Notifications sent on success/failure  

---

## 🪜 Step-by-Step Implementation

### **Step 1: Launch two EC2 Instance**

* Create two EC2 instances in same VPC (Default).

# 1.   Jenkins server

   >>For Jenkins setup instructions, I’ve already created detailed documentation [click here](https://github.com/nikiimisal/Project-Jenkins-CI-CD-Setup-and-Build-Process)
* Security Group:
    * Jenkins Server → Port 8080 , 80 , 22

* Install **Java** and **Maven** on Jenkins server  <br>
in jenkins terminal run this commands
```bash
sudo hostnamectl hostname jenkins
sudo apt update
sudo apt install openjdk-17-jdk -y     # (OPTIONAL)
sudo apt install maven -y
java -version
mvn -version
```

>>>There are two steps to install Maven: <br>
i. Go inside the Jenkins server and install it through the terminal using commands.<br>
ii. Or, install it through the Maven plugin in Jenkins.`Maven Integration` plugin<br>
<br>

>>After plugin installing, go to:<br>
Manage Jenkins → Global Tool Configuration → Maven → Add Maven<br>
→ Give it a name (for example, `Maven3`)<br>
→ Choose “Install automatically” or specify the Maven path manually.<br>

# 2.  Tomcat Server

* Security Group:
    * Tomcat Server → Port 8080 , 80 , 22       <br>
      
in tomcat terminal run this commands

```bash
sudo hostnamectl hostname tomcat
sudo apt update
sudo apt install tomcat10 -y
sudo systemctl start tomcat10
sudo systemctl enable tomcat10
java -version                              #if not avalible install it
sudo apt install openjdk-17-jdk -y
```


<p align="center">
  <img src="https://github.com/nikiimisal/Project-Student-App-using-java-CICD-with-Jenkins-GitHub-Maven/blob/main/img/jenkins-tools-config.png?raw=true" width="700" alt="Jenkins Tools Configuration">
</p>

---

### **Step 2: Create GitHub Repository**

* Repo Name: `student-app-java-cicd`   [Repo](https://github.com/nikiimisal/student-app-using-java--CICD-Jenkins)  
* Branch: `main`

#### **Add Webhook**

* Payload URL → `http://<JENKINS_PUBLIC_IP>:8080/github-webhook/`  

<p align="center">
  <img src="https://github.com/nikiimisal/Project-Student-App-using-java-CICD-with-Jenkins-GitHub-Maven/blob/main/img/github-webhook.png?raw=true" width="700" alt="GitHub Webhook Setup">
</p>

---

### **Step 3: Add SSH Credentials in Jenkins**

> Create New Credentials  
1. Navigate → Manage Jenkins → Credentials → Global  
2. Add → **SSH Username with private key**  

   * ID: `java-app-key`  
   * Username: `ubuntu`  
   * Paste your private key  

<p align="center">
  <img src="https://github.com/nikiimisal/Project-Student-App-using-java-CICD-with-Jenkins-GitHub-Maven/blob/main/img/jenkins-credentials.png?raw=true" width="700" alt="SSH Credentials">
</p>

---

### **Step 4: Create Jenkins Pipeline Job**

* Item Name: `student-app-java-cicd`  
* Item-Type: **Pipeline**  
* Enable Trigger: *GitHub hook trigger for GITScm polling*  
* Definition: *Pipeline script from SCM*  
* Repository URL: `https://github.com/nikiimisal/student-app-using-java--CICD-Jenkins.git`  
* Branch: `main`  
* Script Path: `jenkinsfile`  

<p align="center">
  <img src="https://github.com/nikiimisal/Project-Student-App-using-java-CICD-with-Jenkins-GitHub-Maven/blob/main/img/pipeline-config.png?raw=true" width="700" alt="Jenkins Pipeline Config">
</p>

---

### **Step 5: Jenkinsfile Configuration**

```groovy
pipeline {
    agent any

    environment {
        SERVER_IP    = '13.62.103.164'
        SSH_CRED_ID  = 'node-app-key'
        TOMCAT_PATH  = '/var/lib/tomcat10/webapps'
        TOMCAT_SVC   = 'tomcat10'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/nikiimisal/student-app-using-java--CICD-Jenkins.git'
            }
        }

        stage('Build WAR') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                sshagent([SSH_CRED_ID]) {
                    sh """
                        WAR_FILE=\$(ls target/*.war | head -n 1)
                        scp -o StrictHostKeyChecking=no \$WAR_FILE ubuntu@${SERVER_IP}:/tmp/
                        ssh -o StrictHostKeyChecking=no ubuntu@${SERVER_IP} '
                            sudo rm -rf ${TOMCAT_PATH}/*
                            sudo mv /tmp/*.war ${TOMCAT_PATH}/ROOT.war
                            sudo chown tomcat:tomcat ${TOMCAT_PATH}/ROOT.war
                            sudo systemctl restart ${TOMCAT_SVC}
                        '
                    """
                }
            }
        }
    }

    post {
        success {
            echo "App deployed! Visit: http://${SERVER_IP}:8080/"
        }
        failure {
            echo "Deployment failed."
        }
    }
}
```
>NOTE: Replace `SERVER_IP`, `SSH_CREDENTIAL`, and paths as per your setup.

---

### **Step 6: Push Code to GitHub**

```bash
git init
git add .
git commit -m ""
git push -u origin main
```

>Now when code is pushed, GitHub Webhook triggers Jenkins → Jenkins pulls latest code → builds with Maven → tests → packages → deploys automatically.

---

### **Step 7: Verify Jenkins Build**

* Open Jenkins Dashboard  
* Click on **Build Now**  
* Watch the **Pipeline Stage View**  

<p align="center">
  <img src="https://github.com/nikiimisal/Project-Student-App-using-java-CICD-with-Jenkins-GitHub-Maven/blob/main/img/build-success.png?raw=true" width="700" alt="Pipeline Success">
</p>

>If build fails, check the **Console Output** to find and fix errors.

---

### **Step 8: Verify Application**

Open in browser:  
If deployed on Tomcat →  
```
http://<Your-Tomcat-Server-IP>:8080/student-app
```

If it’s a jar-based app:  
```bash
java -jar target/student-app.jar
```

<p align="center">
  <img src="https://github.com/nikiimisal/Project-Student-App-using-java-CICD-with-Jenkins-GitHub-Maven/blob/main/img/app-running.png?raw=true" width="700" alt="Java App Running">
</p>

---

## 🧠 Troubleshooting

| Issue | Solution |
|-------|-----------|
| Webhook not triggering | Ensure Jenkins Public IP accessible from GitHub |
| Maven not found | Check Jenkins → Global Tool Config |
| SSH Permission denied | Verify private key and username |
| Jenkins build fails | Check Jenkins console log |
| Tomcat not restarting | Restart manually or check service logs |

---

## 🎯 Conclusion

By integrating **Jenkins** with **GitHub Webhooks** and **Maven**, this setup enables **fully automated Java application deployments**.  
Every code push triggers Jenkins to build, test, and deploy — resulting in faster and more reliable delivery cycles.

✅ **Key Benefits:**
* Full CI/CD automation  
* Continuous Integration and Delivery  
* Reduced manual intervention  
* Repeatable, reliable builds  

---

**Author:** Nikhil Dipak Misal 😎  
**License:** MIT  

---

<p align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=0:0072ff,100:00c6ff&height=120&section=footer"/>
</p>
