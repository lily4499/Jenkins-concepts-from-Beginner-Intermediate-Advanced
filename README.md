# Jenkins-concepts-from-Beginner-Intermediate-Advanced
---
Let’s create a **simple Node.js application** from scratch that can be used across all Jenkins demos (beginner to advanced). This app will include:
```
✅ Node.js backend
✅ Basic testing
✅ Dockerfile
✅ Kubernetes manifests
✅ SonarQube-friendly code
✅ Deployment-ready for Minikube, EKS, or GitOps
✅ Proper folder structure for Jenkins usage
```
---

## 📦 Project Name: `node-jenkins-demo-app`

---

## 📁 Folder Structure

```
node-jenkins-demo-app/
├── Jenkinsfile
├── Dockerfile
├── app.js
├── package.json
├── test/
│   └── app.test.js
├── k8s/
│   ├── deployment.yaml
│   └── service.yaml
├── sonar-project.properties
└── README.md
```

---

## 📄 app.js

```js
const express = require('express');
const app = express();

app.get('/', (req, res) => res.send('<h1>Hello from Jenkins Demo App!</h1>'));
app.get('/health', (req, res) => res.status(200).send('OK'));

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`App running on port ${PORT}`));

module.exports = app;
```

---

## 📄 package.json

```json
{
  "name": "node-jenkins-demo-app",
  "version": "1.0.0",
  "description": "Node.js app for Jenkins CI/CD demo",
  "main": "app.js",
  "scripts": {
    "start": "node app.js",
    "test": "jest"
  },
  "author": "LiliTech",
  "license": "MIT",
  "dependencies": {
    "express": "^4.18.2"
  },
  "devDependencies": {
    "jest": "^29.0.0",
    "supertest": "^6.0.0"
  }
}
```

---

## 📄 test/app.test.js

```js
const request = require('supertest');
const app = require('../app');

describe('GET /', () => {
  it('should return 200 and welcome message', async () => {
    const res = await request(app).get('/');
    expect(res.statusCode).toEqual(200);
    expect(res.text).toContain('Hello');
  });
});
```

---

## 📄 Dockerfile

```Dockerfile
FROM node:18

WORKDIR /app
COPY . .
RUN npm install

EXPOSE 3000
CMD ["npm", "start"]
```

---

## 📄 Jenkinsfile

(For demo purposes, uses build + test + Docker + push + deploy steps)

```groovy
pipeline {
  agent any

  parameters {
    string(name: 'TAG', defaultValue: '1.0.0', description: 'Image Tag')
  }

  environment {
    IMAGE = "laly9999/node-jenkins-demo-app"
  }

  stages {
    stage('Install') {
      steps {
        sh 'npm install'
      }
    }

    stage('Test') {
      steps {
        sh 'npm test'
      }
    }

    stage('Run Sonarqube') {
      environment {
          scannerHome = tool 'sonar-scan';
      }
      steps {
          withSonarQubeEnv('MySonar') {
              sh """
                  ${scannerHome}/bin/sonar-scanner \
                 // -Dsonar.projectKey=key-test
              """
            }
        }
    }

    stage('Build Docker Image') {
      steps {
        sh 'docker build -t $IMAGE:${params.TAG} .'
      }
    }

    stage('Push Image') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
          sh 'echo $PASS | docker login -u $USER --password-stdin'
          sh 'docker push $IMAGE:${params.TAG}'
        }
      }
    }

    stage('Deploy to K8s') {
      steps {
        sh 'kubectl apply -f k8s/'
      }
    }
  }

  post {
    always {
      cleanWs()
    }
  }
}
```

---

## 📄 k8s/deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: node-app
  labels:
    app: node-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: node-app
  template:
    metadata:
      labels:
        app: node-app
    spec:
      containers:
        - name: node-app
          image: laly9999/node-jenkins-demo-app:1.0.0
          ports:
            - containerPort: 3000
```

---

## 📄 k8s/service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: node-service
spec:
  type: NodePort
  selector:
    app: node-app
  ports:
    - port: 80
      targetPort: 3000
      nodePort: 30036
```

---

## 📄 sonar-project.properties

```properties
sonar.projectKey=node-jenkins-demo-app
sonar.sources=.
sonar.tests=./test
sonar.inclusions=**/*.js
sonar.test.inclusions=test/**/*.test.js
sonar.javascript.lcov.reportPaths=coverage/lcov.info
```

---

## Quick Start

```bash
npm install
npm test
docker build -t node-jenkins-demo-app .
docker run -p 3000:3000 node-jenkins-demo-app
````

## Run Tests

```bash
npm test
```

## Deploy to Minikube

```bash
kubectl apply -f k8s/
```


---
---

# 🟢 BEGINNER CONCEPTS 

---

### ✅ 1. Jenkins Installation (Docker)

**Demo Steps:**

```bash
# Step 1: Create a volume for persistence
docker volume create jenkins_home

# Step 2: Run Jenkins
docker run -d --name jenkins -p 9090:8080 -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts
```

* Visit: [http://localhost:9090](http://localhost:9090)
* Initial password: `cat /var/jenkins_home/secrets/initialAdminPassword` (inside container)

---

### ✅ 2. Jenkins Dashboard

**Demo:**

* Open Jenkins UI → Dashboard
* Click “New Item” → Name: `demo-job` → Select “Pipeline” → OK

---

### ✅ 3. Create Freestyle Job

**Demo:**

* Dashboard → New Item → “demo-freestyle”
* Choose “Freestyle project”
* Add Shell command:

```bash
echo "Hello from Jenkins Freestyle Job"
```

* Save → Build Now → See Console Output

---

### ✅ 4. Create Pipeline Job with Jenkinsfile

**Demo:**

* Create `Jenkinsfile`:

```groovy
pipeline {
  agent any
  stages {
    stage('Hello') {
      steps {
        echo 'Hello from Jenkins Pipeline'
      }
    }
  }
}
```

* In GitHub repo OR copy/paste into Jenkins job
* Jenkins UI → New Item → “demo-pipeline” → Pipeline → Add Jenkinsfile → Save → Build Now

---

### ✅ 5. Connect to GitHub Repo

**Demo:**

* Jenkins UI → Job Config → Pipeline → Definition: “Pipeline from SCM” → Git
* Paste GitHub repo URL
* Add SSH or HTTPS credentials
* Save → Build Now (or trigger on push)

---

### ✅ 6. Build Triggers (e.g., Webhook)

**Demo:**

* Jenkins UI → Configure Job → Build Triggers → Check “GitHub hook trigger for GITScm polling”
* GitHub → Settings → Webhooks → URL: `http://<jenkins>:8080/github-webhook/`
* Push to GitHub → Jenkins build triggers automatically

---

### ✅ 7. Workspace and Console Output

**Demo:**

* Jenkins UI → Build History → Click a build
* Click **Workspace** → view files
* Click **Console Output** → view logs

---

### ✅ 8. Basic Environment Variable

**Demo:**

```groovy
pipeline {
  agent any
  environment {
    NAME = 'Liliane'
  }
  stages {
    stage('Greet') {
      steps {
        echo "Hello, ${env.NAME}"
      }
    }
  }
}
```

---

### ✅ 9. Basic Shell Command in Pipeline

**Demo:**

```groovy
pipeline {
  agent any
  stages {
    stage('Shell') {
      steps {
        sh 'date'
      }
    }
  }
}
```

---

### ✅ 10. Post-Build Actions (Email, Archive)

**Demo:**

* Jenkins UI → Configure Job → Add “Editable Email Notification”
* Or use this in Jenkinsfile:

```groovy
post {
  success {
    echo 'Build Successful'
  }
  failure {
    echo 'Build Failed'
  }
}
```

---
---

# 🟡 INTERMEDIATE JENKINS CONCEPTS

---

### ✅ 11. Jenkins Plugins

**Purpose:** Extend Jenkins functionality (Docker, Slack, SonarQube, etc.)

**Demo:**
- UI: **Manage Jenkins → Plugins → Available → Install “Docker Pipeline”, “Slack”, “SonarQube Scanner”**

**CLI:**
```bash
docker exec -it jenkins bash
jenkins-plugin-cli --plugins docker-plugin sonar docker-workflow
```

---

### ✅ 12. Jenkins Credentials

**Purpose:** Securely store secrets like GitHub tokens or DockerHub credentials.

**Demo:**
- UI: **Manage Jenkins → Credentials → (global) → Add Credentials**
  - Username/password: `dockerhub`
  - Secret text: GitHub token, Slack webhook

**Use in Jenkinsfile:**
```groovy
withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
  sh 'docker login -u $USER -p $PASS'
}
```

---

### ✅ 13. Parameterized Builds

**Purpose:** Let users input parameters like app version or target environment.

**Demo Jenkinsfile:**
```groovy
pipeline {
  agent any
  parameters {
    string(name: 'APP_VERSION', defaultValue: '1.0.0', description: 'Enter app version')
  }
  stages {
    stage('Print Version') {
      steps {
        echo "App version: ${params.APP_VERSION}"
      }
    }
  }
}
```

**Run:**
- Click “Build with Parameters” in Jenkins UI

---

### ✅ 14. Docker Integration (Build & Push)

**Purpose:** Build Docker image and push to DockerHub

**Dockerfile:**
```dockerfile
FROM node:18
WORKDIR /app
COPY . .
RUN npm install
CMD ["npm", "start"]
```

**Jenkinsfile:**
```groovy
pipeline {
  agent any
  environment {
    IMAGE = "laly9999/node-app"
  }
  parameters {
    string(name: 'TAG', defaultValue: '1', description: 'Docker image tag')
  }
  stages {
    stage('Docker Build') {
      steps {
        sh 'docker build -t $IMAGE:${params.TAG} .'
      }
    }
    stage('Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
          sh 'echo $PASS | docker login -u $USER --password-stdin'
          sh 'docker push $IMAGE:${params.TAG}'
        }
      }
    }
  }
}
```

---

### ✅ 15. SonarQube Integration

**Purpose:** Run static code analysis and quality gates
 Install SonarQube in Docker Container
```docker run -d --name sonarqube \
  -p 9000:9000 \
  sonarqube:lts
```

**Demo:**
- Install SonarQube and Sonar Scanner plugins
- UI: **Manage Jenkins → Configure System → SonarQube servers**
- Add SonarQube token as credentials

**Jenkinsfile:**
```groovy
stage('SonarQube Analysis') {
  steps {
    withSonarQubeEnv('MySonarQubeServer') {
      sh 'sonar-scanner -Dsonar.projectKey=node-app -Dsonar.sources=. -Dsonar.host.url=http://localhost:9000'
    }
  }
}
```

---

### ✅ 16. Slack Notifications

**Purpose:** Notify build status via Slack

**Demo:**
- Install Slack plugin
- Create app + webhook URL in Slack
- Jenkins → Manage Jenkins → Configure System → Slack
- Add credentials and default channel

**Jenkinsfile:**
```groovy
post {
  always {
    slackSend(channel: '#ci-cd', message: "Pipeline Result: ${currentBuild.result}")
  }
}
```

---

### ✅ 17. Archive Artifacts

**Purpose:** Save build output (e.g., `.zip`, `.war`, `.jar`) for download

**Jenkinsfile:**
```groovy
stage('Archive') {
  steps {
    sh 'zip -r output.zip .'
    archiveArtifacts artifacts: 'output.zip', fingerprint: true
  }
}
```

---

### ✅ 18. Pipeline Input (Manual Approval)

**Purpose:** Pause pipeline and wait for human input

**Jenkinsfile:**
```groovy
stage('Approval') {
  steps {
    input message: 'Approve deployment to production?'
  }
}
```

---

### ✅ 19. Parallel Steps

**Purpose:** Run multiple stages simultaneously to reduce build time

**Jenkinsfile:**
```groovy
stage('Parallel Tasks') {
  parallel {
    stage('Test') {
      steps {
        sh 'npm test'
      }
    }
    stage('Lint') {
      steps {
        sh 'npm run lint'
      }
    }
  }
}
```

---

### ✅ 20. Blue Ocean / Stage View

**Purpose:** Visual pipeline UI with status per stage

**Demo:**
- Install Blue Ocean plugin
- Access: `http://localhost:9090/blue`

---
---


# 🔴 ADVANCED JENKINS CONCEPTS – Demo from Scratch

---

### ✅ 21. Multibranch Pipeline

**Purpose:** Automatically detect and build branches or pull requests.

**Demo:**

* UI: Jenkins → New Item → “my-multibranch-pipeline” → Select **Multibranch Pipeline**
* Under **Branch Sources**:

  * Add GitHub → Provide repo URL and credentials
  * Set "Discover branches" and "Discover PRs" as needed
* Jenkins will scan all branches with a `Jenkinsfile`

**CLI (GitHub side webhook):**

```bash
# Add webhook from GitHub side
# URL: http://<jenkins-url>/github-webhook/
```

---

### ✅ 22. Shared Libraries

**Purpose:** Reuse Groovy code (e.g., build steps, templates) across multiple pipelines.

**Demo:**

* Create GitHub repo (e.g., `jenkins-shared-libs`)

* Structure:

  ```
  jenkins-shared-libs/
  └── vars/
      └── buildApp.groovy
  ```

* `buildApp.groovy`:

  ```groovy
  def call() {
    sh 'npm install'
    sh 'npm test'
  }
  ```

* In consuming project’s `Jenkinsfile`:

  ```groovy
  @Library('jenkins-shared-libs') _
  pipeline {
    agent any
    stages {
      stage('Run') {
        steps {
          buildApp()
        }
      }
    }
  }
  ```

* Jenkins → Manage Jenkins → Configure Global Libraries
  Add your shared library → SCM (GitHub) → default version = `master`

---

### ✅ 23. Jenkins Agents (Nodes)

**Purpose:** Distribute build workloads to other machines (e.g., Docker, Windows, etc.)

**Demo:**

* Provision new agent machine or container with Java
* Generate SSH key → Add to agent machine
* Jenkins → Manage Jenkins → Nodes → New Node

  * Launch method: SSH
  * Host: IP/hostname of the agent

**On the agent:**

```bash
sudo apt update
sudo apt install openjdk-11-jre -y
```

**Use in Jenkinsfile:**

```groovy
pipeline {
  agent { label 'docker-agent' }
  stages {
    stage('Run') {
      steps {
        sh 'hostname'
      }
    }
  }
}
```

---

### ✅ 24. Label-Based Agent Assignment

**Purpose:** Run specific stages on certain agents

**Jenkinsfile:**

```groovy
pipeline {
  agent none
  stages {
    stage('Docker Build') {
      agent { label 'docker-agent' }
      steps {
        sh 'docker build -t app:latest .'
      }
    }
    stage('Deploy') {
      agent { label 'k8s-agent' }
      steps {
        sh 'kubectl apply -f deployment.yaml'
      }
    }
  }
}
```

---

### ✅ 25. Kubernetes Deployment

**Purpose:** Automate deployment of apps to Kubernetes clusters (e.g., Minikube, EKS, AKS)

**Demo:**

* Install `kubectl` in Jenkins agent
* Jenkinsfile:

```groovy
pipeline {
  agent any
  environment {
    KUBECONFIG = credentials('kubeconfig-secret') // or mount manually
  }
  stages {
    stage('Deploy to K8s') {
      steps {
        sh 'kubectl apply -f k8s/deployment.yaml'
        sh 'kubectl rollout status deployment/node-app'
      }
    }
  }
}
```

---

### ✅ 26. File Operations (`stash`, `unstash`)

**Purpose:** Share files between stages or agents

**Jenkinsfile:**

```groovy
pipeline {
  agent none
  stages {
    stage('Build') {
      agent any
      steps {
        sh 'npm run build'
        stash includes: '**/dist/**', name: 'built-artifacts'
      }
    }
    stage('Deploy') {
      agent any
      steps {
        unstash 'built-artifacts'
        sh 'kubectl apply -f dist/'
      }
    }
  }
}
```

---

### ✅ 27. GitOps-style Deployment with Jenkins

**Purpose:** Automate deployments by pushing manifests to a GitOps repo

**Demo:**

* Jenkinsfile:

```groovy
stage('GitOps Deployment') {
  steps {
    sh 'git clone https://github.com/org/k8s-deployments.git'
    sh 'cp deployment.yaml k8s-deployments/apps/myapp.yaml'
    dir('k8s-deployments') {
      sh '''
        git config user.email "ci@jenkins"
        git config user.name "jenkins"
        git add .
        git commit -m "Update deployment"
        git push
      '''
    }
  }
}
```

* ArgoCD or Flux will detect this change and deploy it.

---

### ✅ 28. Webhooks and API Triggers

**Purpose:** Trigger builds from GitHub, CLI, or other systems

**CLI (trigger build):**

```bash
curl -X POST "http://localhost:9090/job/my-job/build" \
  --user admin:<api_token>
```

**CLI with parameters:**

```bash
curl -X POST "http://localhost:9090/job/my-job/buildWithParameters" \
  --user admin:<api_token> \
  --data-urlencode "ENV=staging"
```

---

### ✅ 29. Backup & Restore Jenkins

**Purpose:** Save Jenkins config and jobs in case of disaster

**CLI:**

```bash
# Backup Jenkins home
docker cp jenkins:/var/jenkins_home ./jenkins_backup

# Restore
docker stop jenkins
docker rm jenkins
docker run -d --name jenkins -p 9090:8080 -v $(pwd)/jenkins_backup:/var/jenkins_home jenkins/jenkins:lts
```

---

### ✅ 30. Role-Based Access Control (RBAC)

**Purpose:** Control who can access and modify which jobs

**Demo:**

* Install: “Role-based Authorization Strategy” plugin
* UI: Manage Jenkins → Configure Global Security → Enable RBAC
* Manage Jenkins → Manage and Assign Roles

  * Create roles: admin, developer, viewer
  * Assign users/emails to each role

---






