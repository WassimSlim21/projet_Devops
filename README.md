## **1️⃣ Project Overview**
This project automates the software development lifecycle (SDLC) for a **full-stack application** (Spring Boot & Angular). The pipeline ensures:
- **Continuous Integration (CI):** Automated builds, tests, and quality analysis.
- **Continuous Deployment (CD):** Automatic deployment using Docker.

**🛠 Technologies Used:**
| Component        | Technology Used |
|-----------------|----------------|
| **Version Control** | Git & GitHub |
| **Build Tool** | Maven |
| **Testing** | JUnit, Mockito |
| **Code Quality Analysis** | SonarQube |
| **Artifact Repository** | Nexus |
| **Containerization** | Docker & Docker Hub |
| **CI/CD Automation** | Jenkins |
| **Monitoring** | Prometheus & Grafana |
| **Virtualization** | Vagrant (Ubuntu) |

---

## **2️⃣ CI/CD Pipeline Workflow**
### **(A) CI: Continuous Integration**
1. **Jenkins pulls the latest code** from **GitHub**.
2. **Build & Test Backend (Spring Boot)**
   - **Maven** compiles the code.
   - Runs unit tests with **JUnit & Mockito**.
   - **SonarQube** performs **static code analysis** to check for vulnerabilities & code smells.
   - If tests & analysis succeed, Jenkins proceeds.
3. **Package & Store Artifacts**
   - The final **JAR file** is uploaded to **Nexus Repository**.

---

### **(B) CD: Continuous Deployment**
4. **Build Docker Images**
   - **Backend:** A Docker image is created for the **Spring Boot** JAR.
   - **Frontend:** Angular application is built and containerized.
   - **Database:** MySQL is used for data storage.
   - All images are stored in **Docker Hub**.

5. **Deploy Application using Docker Compose**
   - Jenkins triggers **Docker Compose** to launch:
     - **Spring Boot Backend (Port: 8089)**
     - **Angular Frontend (Port: 4201)**
     - **MySQL Database**
   - The containers start inside the **Vagrant Virtual Machine**.

6. **Application Monitoring**
   - **Prometheus** collects metrics from Spring Boot & system logs.
   - **Grafana** visualizes monitoring dashboards.

7. **Automated Email Notifications**
   - Jenkins sends emails with the build and deployment status.

---

## **3️⃣ Infrastructure Setup in Vagrant**
Your project runs inside a **Vagrant-managed Ubuntu virtual machine**. Here's how it's structured:

- **Vagrant provisions the VM** and installs required dependencies.
- Jenkins, Docker, Nexus, and SonarQube run as services.
- Docker manages the deployment.

**📜 Sample `Vagrantfile`:**
```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"
  config.vm.network "private_network", type: "dhcp"
  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get install -y openjdk-11-jdk maven docker.io
    systemctl enable docker
    systemctl start docker
  SHELL
end
```

---

## **4️⃣ Jenkins Pipeline Configuration**
Your Jenkins pipeline follows a **declarative syntax** to automate the entire process.

### **📜 Sample `Jenkinsfile`**
```groovy
pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = credentials('docker-hub-credentials')
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/your-repo.git'
            }
        }

        stage('Build & Test Backend') {
            steps {
                sh 'mvn clean package'
                sh 'mvn test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                sh 'mvn sonar:sonar'
            }
        }

        stage('Publish Artifact to Nexus') {
            steps {
                sh 'mvn deploy'
            }
        }

        stage('Build Docker Images') {
            steps {
                sh 'docker build -t myapp-backend:latest ./backend'
                sh 'docker build -t myapp-frontend:latest ./frontend'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                sh 'docker login -u $DOCKER_HUB_CREDENTIALS_USR -p $DOCKER_HUB_CREDENTIALS_PSW'
                sh 'docker push myapp-backend:latest'
                sh 'docker push myapp-frontend:latest'
            }
        }

        stage('Deploy Application') {
            steps {
                sh 'docker-compose up -d'
            }
        }
    }

    post {
        success {
            mail to: 'admin@example.com', subject: 'Deployment Successful', body: 'Your application is live!'
        }
        failure {
            mail to: 'admin@example.com', subject: 'Deployment Failed', body: 'Check the Jenkins logs for errors.'
        }
    }
}
```

---

## **5️⃣ Docker & Deployment Configuration**
Your `docker-compose.yml` ensures all services start together.

### **📜 Sample `docker-compose.yml`**
```yaml
version: '3.8'
services:
  mysql:
    image: mysql:5.7
    container_name: mysql_db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: mydb
    ports:
      - "3306:3306"

  backend:
    image: myapp-backend:latest
    container_name: backend
    restart: always
    depends_on:
      - mysql
    ports:
      - "8089:8089"

  frontend:
    image: myapp-frontend:latest
    container_name: frontend
    restart: always
    depends_on:
      - backend
    ports:
      - "4201:4201"
```

---

## **6️⃣ Monitoring with Prometheus & Grafana**
- **Prometheus** scrapes metrics from Spring Boot and Docker containers.
- **Grafana** visualizes performance trends.

### **📜 Sample `prometheus.yml`**
```yaml
scrape_configs:
  - job_name: 'spring-boot'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['backend:8089']
```

### **📜 Sample `grafana-datasource.yml`**
```yaml
apiVersion: 1
datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
```

---

## **7️⃣ Automated Notifications**
- **Email alerts:** Jenkins sends notifications about build failures & successes.
- **Slack integration (optional):** Alerts can be pushed to a Slack channel.

---

## **✅ Key Benefits**
✔ **Fully Automated Deployment** – No manual intervention required.  
✔ **Consistent Environments** – Docker ensures identical environments across dev, test, and prod.  
✔ **Scalable & Secure** – Uses **SonarQube**, **Nexus**, and **Docker Hub** to ensure high code quality & artifact management.  
✔ **Efficient Monitoring** – Grafana & Prometheus enable real-time insights into application performance.

---

## **🚀 Next Steps**
Would you like to:
1. **Add security hardening** (SSL, OAuth)?
2. **Enhance logging** with ELK Stack?
3. **Improve rollback mechanisms** in Jenkins?
