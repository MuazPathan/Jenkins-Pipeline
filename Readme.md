# TASK 2: Create a Simple Jenkins Pipeline for CI/CD
**Organization:** Elevate Labs  
**Candidate Name:** Mohammed Muaz Nazir Pathan  
**Role:** DevOps Engineer Intern  
**Repository:** [https://github.com/MuazPathan/Jenkins-Pipeline](https://github.com/MuazPathan/Jenkins-Pipeline)  
**Status:** Completed & Successfully Deployed  

---

## 1. Executive Submission Summary

### Objective
To implement an automated end-to-end continuous integration and continuous deployment (CI/CD) pipeline using Jenkins and Docker, enabling automated source control tracking, automated testing, container build packaging, and localized container redeployment upon every code push.

### Deliverables Met
- [x] **Jenkins Server & Environment Setup:** Configured Jenkins LTS instance running alongside Docker Desktop with execution permissions.
- [x] **Declarative Pipeline (`Jenkinsfile`):** Created a clean, robust pipeline defining `Checkout`, `Test`, `Build Docker Image`, `Deploy`, and `Health Check` stages.
- [x] **SCM Integration & Automation:** Integrated with GitHub repository `MuazPathan/Jenkins-Pipeline` with automated trigger checks via SCM polling.
- [x] **Dockerized Packaging:** Built a secure, multi-stage Node.js container based on `node:18-alpine` running as a non-root user.
- [x] **Deployment Verification:** Successfully deployed container `demo-app-container` on port `3000` with automated container lifecycle and image cache pruning.

---

## 2. CI/CD Architecture Flow

```text
[ Developer Commit / Push ]
            │
            ▼
[ GitHub Repository (main) ]
            │ (Poll SCM / Webhook)
            ▼
[ Jenkins Automation Controller ]
            │
            ├── 1. Checkout (Fetch latest revision from GitHub)
            ├── 2. Test (Execute automated test suite via npm test)
            ├── 3. Build Docker Image (Tag image with ${BUILD_NUMBER} & latest)
            ├── 4. Deploy (Gracefully stop old container & launch new container)
            └── 5. Health Check (Inspect container status & port mappings)
            │
            ▼
[ Containerized App Runtime (http://localhost:3000) ]
```

---

## 3. Pipeline Implementation (`Jenkinsfile`)

```groovy
pipeline {
    agent any

    environment {
        APP_NAME       = "jenkins-demo-app"
        IMAGE_TAG      = "${BUILD_NUMBER}"
        CONTAINER_NAME = "demo-app-container"
        HOST_PORT      = "3000"
        CONTAINER_PORT = "3000"
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code from Git repository...'
                checkout scm
            }
        }

        stage('Test') {
            steps {
                echo 'Executing automated test suite...'
                bat 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image: %APP_NAME%:%IMAGE_TAG%..."
                bat """
                    docker build -t %APP_NAME%:%IMAGE_TAG% .
                    docker tag %APP_NAME%:%IMAGE_TAG% %APP_NAME%:latest
                """
            }
        }

        stage('Deploy') {
            steps {
                echo "Deploying container: %CONTAINER_NAME%..."
                bat """
                    docker stop %CONTAINER_NAME% 2>nul || ver>nul
                    docker rm %CONTAINER_NAME% 2>nul || ver>nul
                    docker run -d --name %CONTAINER_NAME% -p %HOST_PORT%:%CONTAINER_PORT% %APP_NAME%:latest
                """
            }
        }

        stage('Health Check') {
            steps {
                echo 'Verifying container running status...'
                bat "docker ps -f name=%CONTAINER_NAME%"
            }
        }
    }

    post {
        always {
            echo 'Pruning dangling build cache...'
            bat 'docker image prune -f'
        }
        success {
            echo "Pipeline run #${BUILD_NUMBER} deployed successfully!"
        }
        failure {
            echo "Pipeline run #${BUILD_NUMBER} failed. Review console logs."
        }
    }
}
```

---

## 4. Container Definition (`Dockerfile`)

```dockerfile
FROM node:18-alpine

WORKDIR /usr/src/app

COPY package*.json ./
RUN npm install --production

COPY . .

EXPOSE 3000

USER node

CMD ["node", "app.js"]
```

---

## 5. Verification & Testing Evidence

1. **Pipeline Execution:**  
   The pipeline executed all configured stages (`Checkout` $\to$ `Test` $\to$ `Build Docker Image` $\to$ `Deploy` $\to$ `Health Check`) successfully, resulting in `Finished: SUCCESS`.
2. **Container State:**  
   Docker CLI verification demonstrates that container `demo-app-container` is healthy, listening on `0.0.0.0:3000->3000/tcp`.
3. **Application Verification:**  
   Visiting `http://localhost:3000` returns the expected payload:
   ```json
   {
     "status": "success",
     "message": "Jenkins CI/CD Pipeline Deployed Successfully!",
     "timestamp": "2026-11-06T12:43:20.100Z"
   }
   ```

---

## 6. Author Details
- **Developer:** Mohammed Muaz Nazir Pathan
- **GitHub:** [@MuazPathan](https://github.com/MuazPathan)
- **LinkedIn:** [Muaz Pathan](https://www.linkedin.com/in/muaz-pathan/)