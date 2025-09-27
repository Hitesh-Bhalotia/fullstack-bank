# Full-Stack Bank Application - Jenkins Pipeline

This Jenkins pipeline automates the CI/CD process for a full-stack banking application with security scanning, dependency management, and containerized deployment.

## 📄 Project Attribution

**Important Note**: This project is a fork of the original Full-Stack Bank application created by [raphaelalmeidamartins](https://github.com/raphaelalmeidamartins/fullstack-bank). 

- **Original Repository**: https://github.com/raphaelalmeidamartins/fullstack-bank
- **Fork Repository**: https://github.com/Hitesh-Bhalotia/fullstack-bank
- **My Contribution**: The Jenkins pipeline configuration and CI/CD automation setup documented in this README

All credit for the full-stack banking application code (frontend, backend, and core functionality) goes to the original author. This documentation focuses specifically on the Jenkins pipeline implementation and DevOps automation added to the forked repository.

## 🏗️ Pipeline Overview

This declarative Jenkins pipeline performs the following operations:
- Source code checkout from GitHub
- Security vulnerability scanning (OWASP & Trivy)
- Dependency installation for both frontend and backend
- Containerized deployment using Docker Compose

## 📋 Prerequisites

### Jenkins Requirements
- Jenkins server with pipeline plugin support
- Required Jenkins plugins:
  - Git Plugin
  - NodeJS Plugin
  - OWASP Dependency-Check Plugin
  - Docker Plugin
  - Eclipse Temurin installer

### Tool Configurations
Configure the following tools in Jenkins Global Tool Configuration:

#### JDK Configuration
- **Name**: `jdk`
- **Type**: JDK installation
- **Version**: Java 8 or higher

#### Node.js Configuration
- **Name**: `node`
- **Type**: NodeJS installation
- **Version**: Node.js 14+ (recommended)

#### OWASP Dependency Check
- **Name**: `DC`
- **Type**: Dependency-Check installation
- **Installation**: Automatic installer or manual installation

### System Dependencies
Ensure the following tools are installed on Jenkins agents:
- **Trivy**: Container and filesystem vulnerability scanner
- **Docker**: For containerized deployment
- **Docker Compose**: For multi-container orchestration
- **npm**: Node.js package manager

## 🚀 Pipeline Stages

### 1. Git Checkout
```groovy
git branch: 'main', url: 'https://github.com/Hitesh-Bhalotia/fullstack-bank.git'
```
- **Purpose**: Clones the source code from the main branch
- **Repository**: `https://github.com/Hitesh-Bhalotia/fullstack-bank.git`
- **Branch**: `main`

### 2. OWASP Dependency Scan
```groovy
dependencyCheck additionalArguments: '--scan ./app/backend --disableYarnAudit --disableNodeAudit'
```
- **Purpose**: Scans backend dependencies for known vulnerabilities
- **Tool**: OWASP Dependency Check
- **Scope**: `./app/backend` directory
- **Output**: Generates `dependency-check-report.xml`
- **Configuration**: Disables Yarn and Node audit for faster scanning

### 3. Trivy Filesystem Scan
```groovy
sh "trivy fs ."
```
- **Purpose**: Performs comprehensive filesystem vulnerability scanning
- **Tool**: Trivy security scanner
- **Scope**: Entire project directory
- **Detects**: CVEs, misconfigurations, secrets, and licenses

### 4. Root Dependencies Installation
```groovy
sh "npm install"
```
- **Purpose**: Installs root-level Node.js dependencies
- **Location**: Project root directory

### 5. Backend Dependencies
```groovy
dir('/root/.jenkins/workspace/Bank/app/backend') {
    sh "npm install"
}
```
- **Purpose**: Installs backend-specific dependencies
- **Location**: `app/backend` directory
- **Note**: Uses hardcoded workspace path (consider using `${WORKSPACE}`)

### 6. Frontend Dependencies
```groovy
dir('/root/.jenkins/workspace/Bank/app/frontend') {
    sh "npm install"
}
```
- **Purpose**: Installs frontend-specific dependencies
- **Location**: `app/frontend` directory
- **Note**: Uses hardcoded workspace path (consider using `${WORKSPACE}`)

### 7. Container Deployment
```groovy
sh "npm run compose:up -d"
```
- **Purpose**: Deploys application using Docker Compose
- **Command**: Runs in detached mode (`-d`)
- **Requirement**: `package.json` must contain `compose:up` script

## 📁 Expected Project Structure

```
fullstack-bank/
├── app/
│   ├── backend/
│   │   ├── package.json
│   │   └── [backend source files]
│   └── frontend/
│       ├── package.json
│       └── [frontend source files]
├── package.json          # Root package.json with compose:up script
├── docker-compose.yml     # Docker Compose configuration
└── [other project files]
```

## 🔧 Configuration Requirements

### package.json Scripts
Ensure your root `package.json` contains:
```json
{
  "scripts": {
    "compose:up": "docker-compose up"
  }
}
```

### Jenkins Job Configuration
1. Create a new Pipeline job in Jenkins
2. Configure the pipeline to use this Jenkinsfile
3. Set up webhook or polling for automatic builds

## 🛠️ Installation Guide

### 1. Install Trivy (on Jenkins agent)
```bash
# Ubuntu/Debian
sudo apt-get update
sudo apt-get install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo "deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy

# macOS
brew install trivy
```

### 2. Configure OWASP Dependency Check
- Navigate to Jenkins → Manage Jenkins → Global Tool Configuration
- Add OWASP Dependency-Check installation
- Choose automatic installation or specify installation directory

### 3. Jenkins Plugin Installation
Install required plugins via Jenkins Plugin Manager:
- OWASP Dependency-Check Plugin
- NodeJS Plugin
- Git Plugin
- Docker Plugin
- Eclipse Temurin installer

## ⚠️ Known Issues & Improvements

### Current Issues
1. **Hardcoded Paths**: Pipeline uses hardcoded workspace paths
2. **Parallel Execution**: Sequential execution may be slow

### Recommended Improvements

#### 1. Use Environment Variables
```groovy
stage('Backend') {
    steps {
        dir("${WORKSPACE}/app/backend") {
            sh "npm install"
        }
    }
}
```

#### 2. Parallel Execution
```groovy
stage('Install Dependencies') {
    parallel {
        stage('Backend Dependencies') {
            steps {
                dir("${WORKSPACE}/app/backend") {
                    sh "npm install"
                }
            }
        }
        stage('Frontend Dependencies') {
            steps {
                dir("${WORKSPACE}/app/frontend") {
                    sh "npm install"
                }
            }
        }
    }
}
```

## 📊 Security Scanning Reports

### OWASP Dependency Check
- **Report Location**: `dependency-check-report.xml`
- **Jenkins Integration**: Automatically published to Jenkins
- **View**: Available in job build results

### Trivy Scan
- **Output**: Console logs
- **Coverage**: Files, containers, and infrastructure
- **Integration**: Consider adding report publishing

## 🚀 Deployment

The pipeline deploys the application using Docker Compose in detached mode. Ensure your `docker-compose.yml` includes:
- Frontend service configuration
- Backend service configuration
- Database services (if required)
- Network configuration
- Volume mounts
