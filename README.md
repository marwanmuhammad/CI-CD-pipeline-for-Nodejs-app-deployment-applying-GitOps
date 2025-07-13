
<img width="1536" height="1024" alt="CICD Architecture" src="https://github.com/user-attachments/assets/df8cc24d-082c-4e6d-b6e9-ea74e6843ef5" />

# Overview

This project involves deploying the `Node.js application` from the Node.js GitHub repository 
to a local Kubernetes cluster. The deployment will use `ArgoCD` for continuous delivery. We 
will set up a `Jenkins` pipeline to automate the build and deployment process.

![Architecture](https://github.com/user-attachments/assets/d13e38cc-35a9-41d1-aff5-b81692eedc1c)


# Prerequisites

Vm1:

• Docker • Jenkins

VM2 :

• Minikube • ArgoCD

# Machine for jenkins and Docker 

# install Docker

```bash
sudo apt update
sudo apt install curl
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/docker.gpg
sudo add-apt-repository "deb [arch=$(dpkg --print-architecture)] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt -y install lsb-release gnupg apt-transport-https ca-certificates curl software-properties-common
sudo apt -y install docker-ce docker-ce-cli containerd.io docker-compose-plugin docker-registry
sudo usermod -aG docker $USER
newgrp docker
```

# Install Jenkins on local host

```bash
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update
sudo apt install fontconfig openjdk-17-jre -y
sudo apt install -y jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
```

After writing the Docker File , enter the Jenkins GUI and build this script:

    pipeline {
        agent any
    
        stages {
            stage("Checkout code") {
                steps {
                    script {
                        if (!fileExists('nodejs.org')) {
                            sh 'git clone https://github.com/marwan-mohammad/nodejs.org'
                        }
                        dir('nodejs.org') {
                            sh 'git fetch origin'
                            sh 'git checkout main'
                            sh 'git pull'
                        }
                    }
                }
            }
            stage("Install dependencies") {
                steps {
                    dir('nodejs.org') {
                        sh 'npm ci'
                    }
                }
            }
            stage("Run unit testing") {
                steps {
                    dir('nodejs.org') {
                        sh 'npm run test'
                    }
                }
            }
            stage("Dockerize") {
                steps {
                    dir('nodejs.org') {
                        sh 'docker build -t marwanmuhammad/nodejs.org .'
                    }
                }
            }
            stage("Push Docker image") {
                steps {
                    dir('nodejs.org') {
                        withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                            sh 'echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin'
                            sh 'docker push marwanmuhammad/nodejs.org'
                        }
                    }
                }
            }
        }
    }

make sure to configure your docker-hub credentials, the output will be like this:

![Screenshot from 2024-10-25 17-22-21](https://github.com/user-attachments/assets/1e3be097-ecf8-401f-b6a6-8bac1801d49e)

![Screenshot from 2024-10-25 17-22-40](https://github.com/user-attachments/assets/cc1a81d2-1007-44bc-93f2-29ebfbff8d0d)

![Screenshot from 2024-10-25 17-22-47](https://github.com/user-attachments/assets/7434bd5b-c27a-4664-957c-31035d844d40)

![Screenshot from 2024-10-25 17-23-08](https://github.com/user-attachments/assets/632bc5fa-376f-4d5b-bf9c-f7583c20d028)

![Screenshot from 2024-10-25 17-23-13](https://github.com/user-attachments/assets/efc23a0c-77ce-4f95-84b6-4869f543a8e9)

![Screenshot from 2024-10-25 17-23-27](https://github.com/user-attachments/assets/1929a6e8-1b0b-49af-a1f8-ab95c46d16d7)



# Check your DockerHub

![Screenshot 2024-10-25 194413](https://github.com/user-attachments/assets/03063219-a42e-4037-b610-1bee8bd3e62b)

# Another VM for Minikube and kubectl and Argocd

# install minikube and kubectl 

```bash
Install Kubernetes on Ubuntu/Debian 
#### Install Docker First if Installed Skip these steps
sudo apt update
sudo apt install curl
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/docker.gpg
sudo add-apt-repository "deb [arch=$(dpkg --print-architecture)] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt -y install docker-ce docker-ce-cli containerd.io docker-compose-plugin docker-registry 
sudo usermod -aG docker $USER
newgrp docker

######### Install MiniKube (kubernetes platform)
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb
ls
sudo dpkg -i minikube_latest_amd64.deb

minikube start --driver=docker --nodes=2
sudo snap install kubectl --classic
minikube kubectl -- get pods
kubectl cluster-info
kubectl get pods
minikube addons list
minikube dashboard &
kubectl get nodes
kubectl get pods -A

### install argo cd
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

to get password

kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo

and portforward
kubectl port-forward svc/argocd-server -n argocd 8080:443

```

#  deploy your app  

![Screenshot (749)](https://github.com/user-attachments/assets/bd95ef89-eb71-47f7-87e9-a5068962786a)

![Screenshot (748)](https://github.com/user-attachments/assets/64ac14e5-0dcc-4e4d-b01c-561938b97d0f)


### Congratulations!

![WhatsApp Image 2024-10-01 at 19 36 36_828a14cc](https://github.com/user-attachments/assets/572bfd85-4f92-440c-b84b-c6ca854ab46f)


