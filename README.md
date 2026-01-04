# 🚀 Complete Jenkins CI/CD Pipeline Documentation
## DevSecOps Pipeline with GitOps Integration

---

## 📋 Table of Contents

1. [Pipeline Overview](#overview)
2. [Architecture Diagram](#architecture)
3. [Prerequisites](#prerequisites)
4. [Pipeline Configuration](#pipeline-config)
5. [Stage-by-Stage Breakdown](#stages)
6. [Security Implementation](#security)
7. [Docker Image Strategy](#docker-strategy)
8. [GitOps Integration](#gitops)
9. [Troubleshooting](#troubleshooting)
10. [Best Practices](#best-practices)

---

## 🎯 Pipeline Overview {#overview}

### What This Pipeline Does

This is a **complete DevSecOps CI/CD pipeline** that:
- ✅ Builds Java applications using Maven
- ✅ Scans for secrets and vulnerabilities
- ✅ Runs static code analysis (SonarQube)
- ✅ Creates Docker images with proper versioning
- ✅ Pushes to Docker Hub
- ✅ Updates Kubernetes manifests in GitOps repository
- ✅ Triggers Argo CD for deployment

### Technology Stack

| Component | Technology |
|-----------|-----------|
| **Build Tool** | Apache Maven |
| **Language** | Java 17 |
| **CI/CD** | Jenkins (Agent-based) |
| **Secret Scanning** | GitLeaks |
| **Code Quality** | SonarQube |
| **Vulnerability Scan** | OWASP Dependency Check |
| **Container** | Docker |
| **Registry** | Docker Hub |
| **GitOps** | Argo CD |
| **Orchestration** | Kubernetes |

---

## 🏗️ Architecture Diagram {#architecture}

```
┌─────────────────────────────────────────────────────────────┐
│                     Developer Workflow                       │
│  ─────────────────────────────────────────────────────────  │
│  Developer → Code Change → Push to GitHub                   │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                  Jenkins Pipeline Triggered                  │
│  ─────────────────────────────────────────────────────────  │
│  Webhook or Manual Trigger                                  │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│              Stage 1: Cleanup Workspace                      │
│  ─────────────────────────────────────────────────────────  │
│  • Remove old build artifacts                               │
│  • Clean workspace directory                                │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│              Stage 2: Checkout Source Code                   │
│  ─────────────────────────────────────────────────────────  │
│  • Clone repository from GitHub                             │
│  • Checkout 'main' branch                                   │
│  • Use stored credentials                                   │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│              Stage 3: Secret Scan (GitLeaks)                 │
│  ─────────────────────────────────────────────────────────  │
│  • Scan for hardcoded secrets                               │
│  • Check for API keys, passwords, tokens                    │
│  • Generate JSON report                                     │
│  • ⚠️ Does NOT fail build (exit-code 0)                     │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│              Stage 4: Compile Code                           │
│  ─────────────────────────────────────────────────────────  │
│  • mvn clean compile                                        │
│  • Compiles Java source files                               │
│  • Downloads dependencies                                   │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│              Stage 5: Run Unit Tests                         │
│  ─────────────────────────────────────────────────────────  │
│  • mvn test                                                 │
│  • Executes JUnit tests                                     │
│  • Generates test reports                                   │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│       Stage 6: Static Code Analysis (SonarQube)              │
│  ─────────────────────────────────────────────────────────  │
│  • Analyzes code quality                                    │
│  • Checks for bugs, vulnerabilities, code smells            │
│  • Sends results to SonarQube server                        │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│              Stage 7: Quality Gate                           │
│  ─────────────────────────────────────────────────────────  │
│  • Waits for SonarQube analysis                             │
│  • Checks if quality standards met                          │
│  • ❌ FAILS build if standards not met                      │
│  • Timeout: 5 minutes                                       │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│              Stage 8: Package Artifact                       │
│  ─────────────────────────────────────────────────────────  │
│  • mvn package                                              │
│  • Creates .jar file                                        │
│  • Artifact ready for containerization                      │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│         Stage 9: Build & Push Docker Image                   │
│  ─────────────────────────────────────────────────────────  │
│  • Build image: IMAGE_NAME:FIXED_TAG                        │
│  • Push fixed tag: 1.0.0-BUILD_NUMBER                       │
│  • Push rolling tag: latest                                 │
│  • Registry: Docker Hub                                     │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│         Stage 10: Update GitOps Repository                   │
│  ─────────────────────────────────────────────────────────  │
│  • Clone k8s-argocd repository                              │
│  • Update deployment.yaml with new image tag               │
│  • Commit changes                                           │
│  • Push to main branch                                      │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│              Argo CD Automatic Sync                          │
│  ─────────────────────────────────────────────────────────  │
│  • Detects Git change                                       │
│  • Pulls new manifest                                       │
│  • Applies to Kubernetes cluster                            │
│  • Rolling update with zero downtime                        │
└─────────────────────────────────────────────────────────────┘
```

---

## ✅ Prerequisites {#prerequisites}

### Jenkins Configuration Required

#### 1. Jenkins Agent Setup

```bash
# Agent must have label: 'agent-1'
# Install on agent:
- Docker
- Java 17
- Maven 3.x
- GitLeaks
- Git
```

#### 2. Global Tool Configuration

Navigate to: **Manage Jenkins → Global Tool Configuration**

**JDK Configuration:**
```
Name: java-17
Install automatically: ✓
Version: jdk-17.0.x
```

**Maven Configuration:**
```
Name: maven-3
Install automatically: ✓
Version: 3.9.x
```

#### 3. Jenkins Credentials Setup

Navigate to: **Manage Jenkins → Credentials → System → Global credentials**

| Credential ID | Type | Description | Used For |
|---------------|------|-------------|----------|
| `sonar-token` | Secret text | SonarQube authentication token | Code analysis |
| `github` | Username with password | GitHub personal access token | Clone repos |
| `hn` | Username with password | Docker Hub credentials | Push images |

**How to Create Credentials:**

**A. SonarQube Token (`sonar-token`):**

```bash
# In SonarQube UI:
# My Account → Security → Generate Tokens
# Token Name: jenkins
# Type: Global Analysis Token
# Copy the generated token

# In Jenkins:
# Add Credentials → Secret text
# Secret: <paste-token-here>
# ID: sonar-token
```

**B. GitHub Token (`github`):**

```bash
# In GitHub:
# Settings → Developer settings → Personal access tokens
# Generate new token (classic)
# Scopes: repo (full control)

# In Jenkins:
# Add Credentials → Username with password
# Username: <your-github-username>
# Password: <paste-token-here>
# ID: github
```

**C. Docker Hub Credentials (`hn`):**

```bash
# In Jenkins:
# Add Credentials → Username with password
# Username: <docker-hub-username>
# Password: <docker-hub-password-or-token>
# ID: hn
```

---

## 🔧 Pipeline Configuration {#pipeline-config}

### Complete Jenkinsfile

```groovy
pipeline {
    // Execute on specific agent
    agent { 
        label 'agent-1' 
    }
    
    // Define tools to be used
    tools {
        jdk 'java-17'    // Must match Global Tool Configuration
        maven 'maven-3'   // Must match Global Tool Configuration
    }
    
    // Environment variables available to all stages
    environment {
        // Application identifier
        APP_NAME = "register-app-pipeline"
        
        // Docker Hub username
        DOCKER_USER = "YOUR_DOCKERHUB_USERNAME"
        
        // Full image name
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
        
        // 🔒 FIXED TAG: Unique version for each build
        // Example: 1.0.0-42 (where 42 is BUILD_NUMBER)
        FIXED_TAG = "1.0.0-${BUILD_NUMBER}"
        
        // 🔄 ROLLING TAG: Always points to latest
        ROLLING_TAG = "latest"
        
        // SonarQube token from credentials
        SONAR_TOKEN = credentials('sonar-token')
    }
    
    stages {
        // ... stages defined below
    }
}
```

---

## 📊 Stage-by-Stage Breakdown {#stages}

---

### Stage 1: Cleanup Workspace

```groovy
stage("Cleanup Workspace") {
    steps {
        cleanWs()
    }
}
```

**Purpose:**
- Removes all files from previous builds
- Ensures clean slate for new build
- Prevents old artifacts from interfering

**What Happens:**
```bash
# Jenkins executes
rm -rf /var/lib/jenkins/workspace/YOUR_JOB/*
```

**Why Important:**
- Prevents stale files
- Reduces disk usage
- Ensures reproducible builds

---

### Stage 2: Checkout Source Code

```groovy
stage("Checkout Source Code") {
    steps {
        git branch: 'main',
            credentialsId: 'github',
            url: 'https://github.com/YOUR_USERNAME/ci_cd_automation.git'
    }
}
```

**Purpose:**
- Clones the application source code
- Checks out specified branch
- Uses stored GitHub credentials

**What Happens:**
```bash
# Behind the scenes
git clone https://TOKEN@github.com/YOUR_USERNAME/ci_cd_automation.git
cd ci_cd_automation
git checkout main
```

**Configuration Details:**

| Parameter | Value | Description |
|-----------|-------|-------------|
| `branch` | `main` | Branch to build from |
| `credentialsId` | `github` | Jenkins credential ID |
| `url` | GitHub repo URL | Source repository |

**Security Note:**
- ✅ Token never appears in logs
- ✅ Credentials managed by Jenkins
- ✅ No hardcoded passwords

---

### Stage 3: Secret Scan (GitLeaks)

```groovy
stage("Secret Scan (GitLeaks)") {
    steps {
        sh '''
            gitleaks detect \
                --source . \
                --no-git \
                --verbose \
                --report-format json \
                --report-path gitleaks-report.json \
                --exit-code 0
        '''
    }
}
```

**Purpose:**
- Scans code for hardcoded secrets
- Detects API keys, passwords, tokens
- Prevents credential leaks

**Command Breakdown:**

| Flag | Purpose |
|------|---------|
| `--source .` | Scan current directory |
| `--no-git` | Don't use Git history |
| `--verbose` | Detailed output |
| `--report-format json` | JSON output |
| `--report-path` | Save to file |
| `--exit-code 0` | ⚠️ Don't fail build |

**What It Detects:**
```bash
# Examples of secrets GitLeaks finds:
AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
DATABASE_PASSWORD=super_secret_123
PRIVATE_KEY=-----BEGIN RSA PRIVATE KEY-----
api_token: ghp_1234567890abcdef
```

**Report Output (`gitleaks-report.json`):**
```json
{
  "findings": [
    {
      "Description": "AWS Access Key",
      "File": "src/config.java",
      "Line": 42,
      "Secret": "AKIAIOSFODNN7EXAMPLE"
    }
  ]
}
```

**Why `exit-code 0`?**
- Allows build to continue
- Team can review findings
- Doesn't block urgent deployments
- Consider changing to `1` in production

---

### Stage 4: Compile Code

```groovy
stage("Compile Code") {
    steps {
        sh 'mvn clean compile'
    }
}
```

**Purpose:**
- Compiles Java source code to bytecode
- Downloads Maven dependencies
- Validates code syntax

**What Maven Does:**

```bash
# Clean: Remove old compiled files
rm -rf target/

# Compile: Convert .java to .class files
javac src/main/java/**/*.java -d target/classes
```

**Output:**
```
[INFO] Scanning for projects...
[INFO] Building register-app 1.0.0
[INFO] --- maven-compiler-plugin:3.8.1:compile ---
[INFO] Compiling 25 source files to /workspace/target/classes
[INFO] BUILD SUCCESS
```

**Fails If:**
- Syntax errors in code
- Missing dependencies in `pom.xml`
- Java version mismatch

---

### Stage 5: Run Unit Tests

```groovy
stage("Run Unit Tests") {
    steps {
        sh 'mvn test'
    }
}
```

**Purpose:**
- Executes JUnit tests
- Validates business logic
- Ensures code quality

**What Happens:**
```bash
# Maven runs all test classes
mvn test

# Example tests executed:
- UserServiceTest.java
- AuthControllerTest.java
- DatabaseConnectionTest.java
```

**Test Report Generated:**
```
target/
  └── surefire-reports/
      ├── TEST-*.xml
      └── *.txt
```

**Fails If:**
- Any test case fails
- Test coverage below threshold
- Test execution timeout

**Example Test Output:**
```
Tests run: 25, Failures: 0, Errors: 0, Skipped: 0
[INFO] BUILD SUCCESS
```

---

### Stage 6: Static Code Analysis (SonarQube)

```groovy
stage("Static Code Analysis (SonarQube)") {
    steps {
        withSonarQubeEnv('sonarqube') {
            sh '''
                mvn sonar:sonar \
                    -Dsonar.projectKey=register-app \
                    -Dsonar.login=$SONAR_TOKEN
            '''
        }
    }
}
```

**Purpose:**
- Analyzes code quality
- Detects bugs and vulnerabilities
- Checks code coverage
- Identifies code smells

**What Gets Analyzed:**

| Category | What It Checks |
|----------|----------------|
| **Bugs** | Logic errors, null pointers, resource leaks |
| **Vulnerabilities** | SQL injection, XSS, hardcoded credentials |
| **Code Smells** | Duplicate code, complex methods, naming issues |
| **Coverage** | Test coverage percentage |
| **Security Hotspots** | Encryption, authentication, authorization |

**SonarQube Configuration:**

```yaml
# sonar-project.properties (optional)
sonar.projectKey=register-app
sonar.projectName=Register Application
sonar.sources=src/main/java
sonar.tests=src/test/java
sonar.java.binaries=target/classes
sonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
```

**withSonarQubeEnv Explained:**
- Automatically injects SonarQube server URL
- Provides authentication token
- Configured in: **Manage Jenkins → Configure System → SonarQube servers**

**Example Output:**
```
[INFO] ANALYSIS SUCCESSFUL
[INFO] Published 235 issues
[INFO] Security: 2 vulnerabilities
[INFO] Reliability: 5 bugs
[INFO] Coverage: 78.5%
```

---

### Stage 7: Quality Gate

```groovy
stage("Quality Gate") {
    steps {
        timeout(time: 5, unit: 'MINUTES') {
            waitForQualityGate abortPipeline: true
        }
    }
}
```

**Purpose:**
- Waits for SonarQube analysis to complete
- Checks if quality standards are met
- **Fails build if standards not met**

**Quality Gate Conditions (Example):**

| Metric | Threshold | Action if Failed |
|--------|-----------|------------------|
| Coverage | > 80% | ❌ Build fails |
| Bugs | = 0 | ❌ Build fails |
| Vulnerabilities | = 0 | ❌ Build fails |
| Code Smells | < 10 | ⚠️ Warning |
| Duplications | < 3% | ⚠️ Warning |

**Flow Diagram:**

```
SonarQube Analysis Complete
        ↓
Quality Gate Evaluation
        ↓
    Pass? ──→ ✅ Continue Pipeline
        ↓ No
    ❌ Abort Pipeline
```

**Configuration:**

In SonarQube UI:
1. Quality Gates → Create
2. Set conditions
3. Assign to project

**Timeout Explanation:**
- Maximum wait: 5 minutes
- Prevents indefinite waiting
- Fails build if timeout exceeded

**Build Behavior:**

```groovy
abortPipeline: true
```
- If Quality Gate fails → **Build stops**
- No Docker image created
- No deployment happens

---

### Stage 8: Package Artifact

```groovy
stage("Package Artifact") {
    steps {
        sh 'mvn package'
    }
}
```

**Purpose:**
- Creates executable `.jar` file
- Packages compiled code + dependencies
- Prepares artifact for containerization

**What Maven Does:**

```bash
# Creates JAR file
mvn package

# Output:
target/
  └── register-app-1.0.0.jar  # Executable JAR
```

**JAR Contents:**
```
register-app-1.0.0.jar
├── META-INF/
│   └── MANIFEST.MF
├── com/
│   └── example/
│       └── *.class
├── lib/
│   └── dependencies.jar
└── application.properties
```

**Skips Tests:**

If tests already ran:
```groovy
sh 'mvn package -DskipTests'
```

---

### Stage 9: Build & Push Docker Image

```groovy
stage("Build & Push Docker Image") {
    steps {
        script {
            docker.withRegistry('https://registry.hub.docker.com', 'hn') {
                def image = docker.build("${IMAGE_NAME}:${FIXED_TAG}")
                
                // Push fixed version
                image.push()
                
                // Push rolling tag
                image.push("${ROLLING_TAG}")
            }
        }
    }
}
```

**Purpose:**
- Creates Docker image from JAR
- Tags with two strategies: fixed + rolling
- Pushes to Docker Hub

---

#### Docker Image Build Process

**Assumed Dockerfile:**

```dockerfile
FROM openjdk:17-jdk-alpine
WORKDIR /app
COPY target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Build Command (Behind the Scenes):**

```bash
docker build -t USERNAME/register-app-pipeline:1.0.0-42 .
docker build -t USERNAME/register-app-pipeline:latest .
```

---

#### Tagging Strategy Explained

**1. Fixed Tag: `1.0.0-${BUILD_NUMBER}`**

```
Build #42 → 1.0.0-42
Build #43 → 1.0.0-43
Build #44 → 1.0.0-44
```

**Benefits:**
- ✅ Immutable
- ✅ Traceable to specific build
- ✅ Easy rollback
- ✅ Audit trail

**2. Rolling Tag: `latest`**

```
Always points to most recent build
```

**Benefits:**
- ✅ Simple reference
- ✅ Quick testing
- ⚠️ Not recommended for production

---

#### Registry Authentication

```groovy
docker.withRegistry('https://registry.hub.docker.com', 'hn')
```

**Breakdown:**

| Parameter | Value | Purpose |
|-----------|-------|---------|
| Registry URL | `registry.hub.docker.com` | Docker Hub |
| Credentials ID | `hn` | Jenkins credential |

**What Happens:**

```bash
# Jenkins executes
docker login registry.hub.docker.com \
  -u <username> \
  -p <password>

# Then pushes
docker push USERNAME/register-app-pipeline:1.0.0-42
docker push USERNAME/register-app-pipeline:latest
```

---

#### Complete Image Lifecycle

```
Code → JAR → Docker Build → Tag → Push → Registry
                    ↓
          1.0.0-42 (fixed)
                    ↓
           latest (rolling)
```

**Final Images in Docker Hub:**

```
USERNAME/register-app-pipeline:1.0.0-41
USERNAME/register-app-pipeline:1.0.0-42  ← New
USERNAME/register-app-pipeline:latest    ← Points to 1.0.0-42
```

---

### Stage 10: Update GitOps Repository

```groovy
stage("Update GitOps Repo") {
    steps {
        withCredentials([usernamePassword(
            credentialsId: 'github',
            usernameVariable: 'GIT_USER',
            passwordVariable: 'GIT_TOKEN'
        )]) {
            sh """
                git clone https://${GIT_USER}:${GIT_TOKEN}@github.com/YOUR_USERNAME/k8s-argocd.git
                cd k8s-argocd/apps/register-app
                sed -i 's|image: .*|image: USERNAME/register-app:${FIXED_TAG}|' deployment.yaml
                git add deployment.yaml
                git commit -m "Update image to ${FIXED_TAG}"
                git push origin main
            """
        }
    }
}
```

**Purpose:**
- Updates Kubernetes manifests with new image tag
- Triggers Argo CD sync
- Completes GitOps workflow

---

#### Step-by-Step Breakdown

**1. Clone GitOps Repository**

```bash
git clone https://${GIT_USER}:${GIT_TOKEN}@github.com/YOUR_USERNAME/k8s-argocd.git
```

**Security Note:**
- Token injected at runtime
- Never logged or stored
- Automatically masked in console

**2. Navigate to App Directory**

```bash
cd k8s-argocd/apps/register-app
```

**Expected Structure:**
```
k8s-argocd/
└── apps/
    └── register-app/
        ├── deployment.yaml
        ├── service.yaml
        └── configmap.yaml
```

**3. Update Image Tag**

```bash
sed -i 's|image: .*|image: USERNAME/register-app:1.0.0-42|' deployment.yaml
```

**Before (`deployment.yaml`):**
```yaml
spec:
  containers:
    - name: register-app
      image: USERNAME/register-app:1.0.0-41  # Old
```

**After:**
```yaml
spec:
  containers:
    - name: register-app
      image: USERNAME/register-app:1.0.0-42  # New
```

**4. Commit and Push**

```bash
git add deployment.yaml
git commit -m "Update image to 1.0.0-42"
git push origin main
```

---

#### GitOps Trigger Flow

```
Jenkins updates deployment.yaml
        ↓
Git commit pushed
        ↓
Argo CD detects change (within 3 min)
        ↓
Pulls new manifest
        ↓
Applies to Kubernetes
        ↓
Rolling update
        ↓
Zero-downtime deployment ✅
```

---

## 🔐 Security Implementation {#security}

### 1. Credential Management

**All Secrets Stored in Jenkins:**

| Secret | Type | Storage | Usage |
|--------|------|---------|-------|
| SonarQube Token | Secret Text | Jenkins Credentials | Code analysis |
| GitHub Token | Username/Password | Jenkins Credentials | Git operations |
| Docker Hub Creds | Username/Password | Jenkins Credentials | Image push |

**Security Features:**

```groovy
// ✅ GOOD: Credentials injected at runtime
SONAR_TOKEN = credentials('sonar-token')

// ✅ GOOD: Masked in logs
withCredentials([...]) { ... }

// ❌ BAD: Never do this
PASSWORD = "hardcoded_password"
```

---

### 2. Token Masking in Logs

**Console Output:**

```
# What you see:
git clone https://****:****@github.com/user/repo.git

# Actual command (hidden):
git clone https://username:ghp_token123@github.com/user/repo.git
```

---

### 3. Secret Scanning

**GitLeaks checks for:**

- AWS keys
- API tokens
- Private keys
- Passwords in code
- Database connection strings

**Prevention Strategy:**

1. Scan in pipeline ✅
2. Pre-commit hooks ✅
3. Developer training ✅
4. Use environment variables ✅

---

### 4. SonarQube Security Analysis

**Detects:**

| Vulnerability | Example |
|---------------|---------|
| SQL Injection | `query = "SELECT * FROM users WHERE id=" + userId` |
| XSS | `response.write(userInput)` |
| Hardcoded Secrets | `String apiKey = "abc123"` |
| Weak Crypto | `MD5`, `SHA1` |

---

## 🐳 Docker Image Strategy {#docker-strategy}

### Why Two Tags?

```
USERNAME/register-app:1.0.0-42   ← Fixed (Immutable)
USERNAME/register-app:latest      ← Rolling (Mutable)
```

---

### Fixed Tag Strategy

**Format:** `MAJOR.MINOR.PATCH-BUILD_NUMBER`

**Examples:**
```
1.0.0-1    ← First build
1.0.0-2    ← Second build
1.0.0-42   ← 42nd build
```

**Benefits:**

| Benefit | Description |
|---------|-------------|
| **Traceability** | Know exact build for each deployment |
| **Rollback** | Easy to revert: `image: app:1.0.0-41` |
| **Audit** | Full history of deployments |
| **Immutability** | Tag never changes |

**Production Use:**

```yaml
# Kubernetes deployment.yaml
spec:
  containers:
    - name: app
      image: USERNAME/register-app:1.0.0-42  # Fixed tag
```

---

### Rolling Tag Strategy

**Always:** `latest`

**Benefits:**

| Benefit | Description |
|---------|-------------|
| **Simplicity** | `docker pull app:latest` |
| **Testing** | Quick local testing |
| **CI/CD** | Default tag for dev environments |

**⚠️ Caution:**

```yaml
# ❌ NOT recommended for production
image: USERNAME/register-app:latest

# Why?
# - Can break unexpectedly
# - No version control
# - Hard to debug
# - Rollback difficult
```

**Recommended Use:**

```yaml
# Development/Staging
image: USERNAME/register-app:latest

# Production
image: USERNAME/register-app:1.0.0-42
```

---

### Complete Tagging Flow

```
Build #42 Triggered
        ↓
Docker Build
        ↓
Create Image: register-app:1.0.0-42
        ↓
Tag as: register-app:latest
        ↓
Push Both Tags to Docker Hub
        ↓
GitOps Repo Updated with Fixed Tag (1.0.0-42)
        ↓
Argo CD Deploys Fixed Tag
```

---

## 🔄 GitOps Integration {#gitops}

### GitOps Principles Applied

1. **Declarative:** Entire system state in Git
2. **Versioned:** Full history of changes
3. **Immutable:** Git commits are permanent
4. **Automated:** Argo CD syncs automatically

---

### Repository Structure

```
k8s-argocd/
└── apps/
    └── register-app/
        ├── deployment.yaml     ← Updated by Jenkins
        ├── service.yaml
        ├── configmap.yaml
        ├── secret.yaml
        └── ingress.yaml
```

---

### Deployment Flow

```
Developer Push
      ↓
Jenkins CI
      ↓
Docker Build
      ↓
Update GitOps Repo (deployment.yaml)
      ↓
Git Commit
      ↓
Argo CD Detects Change
      ↓
Kubernetes Sync
      ↓
Rolling Update
      ↓
Zero-Down
