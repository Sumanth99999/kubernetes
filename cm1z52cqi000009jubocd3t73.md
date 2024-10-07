---
title: "Ultimate CI/CD Pipeline Using Jenkins, Maven, SonarQube, Docker, Kubernetes, ArgoCD, and Helm"
datePublished: Mon Oct 07 2024 15:00:16 GMT+0000 (Coordinated Universal Time)
cuid: cm1z52cqi000009jubocd3t73
slug: ultimate-cicd-pipeline-using-jenkins-maven-sonarqube-docker-kubernetes-argocd-and-helm
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1728312995730/b32bc2f3-7bd8-4e77-ad60-d588df42b22f.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1728313200768/80c434b8-782b-4469-9765-7c48126e456c.png

---

This guide will walk you through setting up an ultimate CI/CD pipeline for a Java application, built with Spring Boot, using Jenkins as the orchestrator. We’ll integrate Maven for building, SonarQube for static code analysis, Docker for containerization, Kubernetes for deployment, Argo CD for continuous delivery, and Helm for managing Kubernetes deployments.

# Spring Boot based Java web application

This is a simple Sprint Boot based Java application that can be built using Maven. Sprint Boot dependencies are handled using the pom.xml at the root directory of the repository.

This is a MVC architecture based application where controller returns a page with title and message attributes to the view.

## Execute the application locally and access it using your browser

Checkout the repo and move to the directory

```plaintext
git clone https://github.com/iam-veeramalla/Jenkins-Zero-To-Hero/java-maven-sonar-argocd-helm-k8s/sprint-boot-app
cd java-maven-sonar-argocd-helm-k8s/sprint-boot-app
```

Execute the Maven targets to generate the artifacts

```plaintext
mvn clean package
```

The above maven target stroes the artifacts to the `target` directory. You can either execute the artifact on your local machine (or) run it as a Docker container.

**Note: To avoid issues with local setup, Java versions and other dependencies, I would recommend the docker way.**

Output:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728311434122/92b8ab9a-6d74-4896-8984-12ec895d70d0.png align="center")

### Execute locally (Java 11 needed) and access the application on [http://localhost:8080](http://localhost:8080/)

```plaintext
java -jar target/spring-boot-web.jar
```

### The Docker way

Build the Docker Image

```plaintext
docker build -t ultimate-cicd-pipeline:v1 .
```

```plaintext
docker run -d -p 8010:8080 -t ultimate-cicd-pipeline:v1
```

Hurray !! Access the application on `http://<ip-address>:8010`

The final Project image is:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728312564180/c0b4a30e-b531-4f40-8c01-965bfbf06d45.png align="center")

## **Prerequisites**

Before diving into the pipeline configuration, ensure you have the following:

* **Java application code**: Hosted in a Git repository (GitHub, GitLab, Bitbucket, etc.).
    
* **Ola Cloud**: CI part of the pipeline will be running here.
    
* **Local Kubernetes Cluster**: Minikube or a local cluster to manage CD.
    
* **Jenkins**: Installed on Ola Cloud.
    
* **Docker**: Installed both on Ola Cloud and locally for building images.
    
* **SonarQube**: Installed on Ola Cloud for static code analysis.
    
* **Argo CD**: Installed locally for continuous delivery.
    

## **Software Installation Guide**

### **Part 1: CI on Ola Cloud**

#### Step-1. **Install Jenkins:**

Follow these steps to install Jenkins on your server:

* For Ubuntu/Debian:
    
* ```plaintext
    sudo apt update
    sudo apt install openjdk-11-jdk
    wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
    sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
    sudo apt update
    sudo apt install jenkins
    sudo systemctl start jenkins
    sudo systemctl status jenkins
    ```
    

Open your browser and access Jenkins at `http://<your-server-ip>:8080`.

**Step 2: Install Necessary Jenkins Plugins**

1. **Log in to Jenkins** and go to `Manage Jenkins` &gt; `Manage Plugins`.
    
2. **Install the following plugins**:
    
    * Docker Pipeline
        
    * SonarQube Scanner
        

**Step 3: Set Up SonarQube**

**Create a SonarQube user**:

* ```plaintext
    sudo adduser sonarqube
    ```
    

**Switch to the SonarQube user**:

```plaintext
su - sonarqube
```

**Install SonarQube**:

```plaintext
apt install unzip
adduser sonarqube
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.4.0.54424.zip
unzip *
chmod -R 755 /home/sonarqube/sonarqube-9.4.0.54424
chown -R sonarqube:sonarqube /home/sonarqube/sonarqube-9.4.0.54424
cd sonarqube-9.4.0.54424/bin/linux-x86-64/
./sonar.sh start
```

* **Access SonarQube**: Open your browser and go to [`http://your-ola-cloud-ip:9000`](http://your-ola-cloud-ip:9000). The default credentials are:
    
    * Username: `admin`
        
    * Password: `admin`
        
* **Create an API token** for Jenkins to communicate with SonarQube:
    
    * Go to `My Account` &gt; `Security` &gt; `Generate Tokens`.
        
* **Store the API token in Jenkins**:
    
    * Go to `Manage Jenkins` &gt; `Manage Credentials` &gt; `(global)` &gt; `Add Credentials`.
        
    * Select `Secret text`, enter your token, and give it an ID (e.g., `sonarqube`).
        

### **Step-4 : Configure Docker Credentials in Jenkins**

1. **Store Docker Hub credentials**:
    
    * Go to `Manage Jenkins` &gt; `Manage Credentials` &gt; `(global)` &gt; `Add Credentials`.
        
    * Select `Username with password`, enter your Docker Hub username and password, and give it an ID (e.g., `docker-cred`).
        

### **Step-5: Set Up GitHub Token**

1. **Create a GitHub personal access token** with repo permissions:
    
    * Go to GitHub settings &gt; Developer settings &gt; Personal access tokens &gt; Tokens (classic) &gt; Generate new token.
        
2. **Store the token in Jenkins**:
    
    * Go to `Manage Jenkins` &gt; `Manage Credentials` &gt; `(global)` &gt; `Add Credentials`.
        
    * Select `Secret text`, enter your token, and give it an ID (e.g., `github`).
        

### Outputs:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728311575028/66a53810-00ce-4ee1-958a-a712c2b8878c.png align="center")

## **Step-6: Jenkins Pipeline Configuration**

In Ola Cloud, Jenkins will orchestrate the build and test stages of our CI pipeline. We will use the following pipeline stages:

1. **Checkout Code**: Pull the latest code from the Git repository.
    
2. **Build and Test**: Use Maven to compile and run tests.
    
3. **Static Code Analysis**: Analyze code using SonarQube.
    
4. **Build Docker Image**: Containerize the Spring Boot application.
    
5. **Push Docker Image**: Push the image to Docker Hub for deployment.
    
    ```plaintext
    pipeline {
      agent {
        docker {
          image 'abhishekf5/maven-abhishek-docker-agent:v1'
          args '--user root -v /var/run/docker.sock:/var/run/docker.sock' // Mount Docker socket
        }
      }
      stages {
        stage('Checkout') {
          steps {
            git branch: 'main', url: 'https://github.com/<your-repo>/java-maven-sonar-argocd-helm-k8s.git'
          }
        }
        stage('Build and Test') {
          steps {
            sh 'cd java-maven-sonar-argocd-helm-k8s/spring-boot-app && mvn clean package'
          }
        }
        stage('Static Code Analysis') {
          environment {
            SONAR_URL = "http://<sonarqube-ip>:9000"
          }
          steps {
            withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
              sh 'cd java-maven-sonar-argocd-helm-k8s/spring-boot-app && mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
            }
          }
        }
        stage('Build and Push Docker Image') {
          environment {
            DOCKER_IMAGE = "your-docker-repo/cicd:${BUILD_NUMBER}"
            REGISTRY_CREDENTIALS = credentials('docker-cred')
          }
          steps {
            script {
              sh 'cd java-maven-sonar-argocd-helm-k8s/spring-boot-app && docker build -t ${DOCKER_IMAGE} .'
              def dockerImage = docker.image("${DOCKER_IMAGE}")
              docker.withRegistry('https://index.docker.io/v1/', "docker-cred") {
                dockerImage.push()
              }
            }
          }
        }
      }
    }
    ```
    
    After Running the Jenkins Pipe line this is the following output that indicates all the stages get executed
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728311641453/7ce537bb-c98a-4888-bb9d-b1ff3f672228.png align="center")
    
    And we can also check it in by logging in docker hub by searching the image name
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728311709384/c4583dd6-20a6-4cab-b693-5d10a465594a.png align="center")
    
    And we can also see that the build values are also get updated in github
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728311852449/0a3fec93-57cd-46e3-9b2f-7af6e72d96f8.png align="center")
    
    As we can see the artifacts are get updated
    
    ### **Part 2:** Continuous Deployment with Minikube and Argo CD
    
    In this section, we will cover how to set up a Continuous Deployment (CD) pipeline using Minikube and Argo CD. This setup allows for automated deployments of your Java application whenever there are changes in the CI pipeline.
    
    ## **Prerequisites**
    
    Before starting, ensure you have the following:
    
    1. A local machine with Minikube installed.
        
    2. Access to the CI pipeline that we set up previously.
        
    3. `kubectl` installed and configured to communicate with your Minikube cluster.
        
    
    ## **Step 1: Install Minikube**
    
    1. **Install Minikube**: You can install Minikube using the following commands. Make sure to have `kubectl` and a hypervisor installed (like VirtualBox or Docker).
        
        ```plaintext
        # Install Minikube on Linux
        curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
        sudo install minikube-linux-amd64 /usr/local/bin/minikube
        
        # Start Minikube
        minikube start
        ```
        
        Step 2: Install Argo CD
        
        Install Operator Lifecycle Manager (OLM), a tool to help manage the Operators running on your cluster.
        
        ```plaintext
        curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.28.0/install.sh | bash -s v0.28.0
        ```
        
        Install the operator by running the following command:
        
        ```plaintext
        kubectl create -f https://operatorhub.io/install/argocd-operator.yaml
        ```
        
        After install, watch your operator come up using next command.
        
        ```plaintext
        kubectl get csv -n operators
        ```
        
        ### Configure Argo CD Controller
        
        Creating an Argocd Controller:
        
        ```plaintext
        apiVersion: argoproj.io/v1alpha1
        kind: ArgoCD
        metadata:
            name: example-argocd
            labels:
              example: basic
        spec: {}                                                                                                                    ~    
        ```
        
        Apply the controller
        
        ```plaintext
        kubectl apply -f argocd.yml
        ```
        
        Output:
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728312050341/b8d761db-6da8-49c6-918e-2abbd43a40c3.png align="center")
        
        **Expose the Argo CD server**: By default, the Argo CD server runs on a ClusterIP service. We will change it to NodePort to access it externally.
        
        ```plaintext
        kubectl edit svc example-argocd-server
        ```
        

Find the line that starts with `type: ClusterIP` and change it to `type: NodePort`. Save the changes.

### **Get the NodePort**:

### You can get the port assigned by running:

```plaintext
minikube service example-argocd-server
```

Note down the NodePort assigned to the Argo CD server.

Output:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728312202224/343d23b7-2cde-4f36-a150-f616066dcb88.png align="center")

### **Acess the Argo CD UI**:

Open your browser and navigate to `http://<MINIKUBE_IP>:<NODE_PORT>`. You can get the Minikube IP using:

The Default username and Password for Argo cd is :

Username : admin

By default Argo cd’s passwords are stored in secrets, we can acess them by

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728312719651/670f96b6-7a8f-4c7d-b410-6ad4074343bc.png align="center")

## **Step 4: Integrate CI with CD**

1. **Create an Application in Argo CD**:
    
    * Log in to the Argo CD UI.
        
    * Click on the **\+ NEW APP** button to create a new application.
        
    * Fill out the form as follows:
        
        * **Application Name**: Choose a name for your application.
            
        * **Project**: Default
            
        * **Sync Policy**: Set to **Automated** for automatic syncing.
            
        * **Repository URL**: Provide the CI part Git repository URL.
            
        * **Path**: Specify the path to your Kubernetes manifests (e.g., `java-maven-sonar-argocd-helm-k8s/spring-boot-app-manifests`).
            
        * **Cluster**: Select the cluster where you want to deploy (default is your Minikube cluster).
            
        * **Namespace**: Specify the Kubernetes namespace to deploy.
            
2. **Deploy the Application**: After creating the application, Argo CD will automatically sync and deploy your application whenever there are changes in the specified repository.
    
3. **Monitor the Deployment**: You can monitor the status of your deployments in the Argo CD UI, which will show the health and sync status of your application.
    

### Output:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728312254969/e63bb1ba-c4da-4791-8915-f0decdf9bf5f.png align="center")

## **Achievements from CI and CD Setup**

Through the implementation of both the Continuous Integration and Continuous Deployment parts of this project, I have achieved several significant milestones:

1. **Automated Build and Test Process**:
    
    * Successfully set up a CI pipeline using Jenkins on an Ola Cloud compute instance, automating the build and testing of a Spring Boot application.
        
    * Integrated Maven for building the Java application and included SonarQube for static code analysis, ensuring code quality and reliability.
        
2. **Efficient Docker Image Management**:
    
    * Created a Dockerfile for packaging the Spring Boot application into a Docker image.
        
    * Automated the process of building and pushing Docker images to Docker Hub, allowing for easy access and deployment of application versions.
        
3. **Version Control and YAML File Management**:
    
    * Implemented a system to automatically update Kubernetes deployment YAML files whenever new images are pushed to Docker Hub, ensuring that the latest versions are always in use.
        
    * Utilized GitHub credentials to modify YAML files programmatically, streamlining the deployment process.
        
4. **Seamless Deployment with Argo CD**:
    
    * Set up a local Kubernetes cluster using Minikube and installed Argo CD to manage deployments.
        
    * Integrated the CI pipeline with Argo CD for automated syncing and deployment of application updates, reducing manual intervention and improving efficiency.
        
5. **Enhanced Monitoring and Management**:
    
    * Leveraged the Argo CD UI for real-time monitoring of application health and deployment status, enabling quick responses to any issues that may arise.
        

## **Conclusion**

In conclusion, this project has successfully integrated Continuous Integration and Continuous Deployment practices, creating a robust pipeline that enhances the software development lifecycle. By automating the build, test, and deployment processes, I have minimized manual tasks, improved code quality, and ensured that applications are consistently delivered to production environments. The use of Jenkins, SonarQube, Docker, Minikube, and Argo CD has not only streamlined development but also fostered a culture of collaboration and rapid iteration.

The skills and knowledge gained from this project have equipped me to tackle more complex deployment scenarios in the future, making me well-prepared for the evolving demands of software development and deployment in cloud environments. This experience has also highlighted the importance of CI/CD practices in delivering high-quality software efficiently and reliably.