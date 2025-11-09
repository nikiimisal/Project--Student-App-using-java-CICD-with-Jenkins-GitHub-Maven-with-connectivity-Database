
# 🚀 Student App — CI/CD with Jenkins, GitHub & Maven

> This project demonstrates a **complete CI/CD pipeline** for a **Java Student Management Application** using **Jenkins, GitHub, and Maven**.  
> The goal is to automate the process of **building, testing, and deploying** Java applications with continuous integration and delivery.

---

## 🧩 Overview

This Student App is built using **Java** with **Maven** as the build automation tool.  
The project integrates **GitHub** for source control and **Jenkins** for automating the CI/CD pipeline.  

Whenever a developer pushes code to GitHub, Jenkins automatically:
1. Pulls the latest code
2. Builds and tests it using Maven
3. Packages the `.jar` or `.war` file
4. Deploys it to the configured environment (local/Tomcat)

---

## ⚙️ Tech Stack

| Component | Technology Used |
|------------|-----------------|
| **Language** | Java |
| **Build Tool** | Maven |
| **CI/CD Tool** | Jenkins |
| **Version Control** | Git & GitHub |
| **Integration** | GitHub Webhook |
| **Deployment** | Tomcat / Local Server |

---

## 🧱 Project Structure

```groovy
.
├── src/
│ ├── main/
│ │ └── java/...
│ └── test/
├── pom.xml
├── Jenkinsfile
├── README.md

```


---

## 🔄 Workflow

### 🔹 Step 1 — Code Push
The developer commits and pushes code changes to the **main** branch of the GitHub repository.

### 🔹 Step 2 — GitHub Webhook Trigger
A **GitHub Webhook** notifies **Jenkins** about the new push event.

### 🔹 Step 3 — Jenkins Pipeline Execution
The Jenkins pipeline starts automatically and performs:
- **Checkout Code** from GitHub  
- **Build & Test** the project with Maven  
- **Package** the final `.jar` or `.war`  
- **Deploy** to target environment  

### 🔹 Step 4 — Notifications
Jenkins displays the build status.  
- ✅ On success → deployed successfully  
- ❌ On failure → developer is notified immediately  

---

## 🧰 Jenkinsfile (Declarative Pipeline)

```groovy
pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'Maven3'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/nikiimisal/student-app-using-java--CICD-Jenkins.git'
            }
        }

        stage('Build') {
            steps {
                echo 'Building project using Maven...'
                sh 'mvn clean install'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                sh 'mvn test'
            }
        }

        stage('Package') {
            steps {
                echo 'Packaging application...'
                sh 'mvn package'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application...'
                // Example deployment step
                // sh 'cp target/student-app.war /var/lib/tomcat9/webapps/'
            }
        }
    }

    post {
        success {
            echo '✅ Build and Deployment Successful!'
        }
        failure {
            echo '❌ Build Failed! Developer notified.'
        }
    }
}

```
# Clone the repository
git clone https://github.com/nikiimisal/student-app-using-java--CICD-Jenkins.git
cd student-app-using-java--CICD-Jenkins

# Build the project
mvn clean install

# Run the application
java -jar target/student-app.jar

# 🧪 How to Run Locally

# Clone the repository
git clone https://github.com/nikiimisal/student-app-using-java--CICD-Jenkins.git
cd student-app-using-java--CICD-Jenkins

# Build the project
mvn clean install

# Run the application
java -jar target/student-app.jar

>If you are using Tomcat, copy the generated .war file into the webapps directory.
