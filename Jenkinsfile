pipeline {
  agent any
  
  tools { nodejs "node" }

  stages {
    stage("Setup EKS Tools") {
      steps {
        script {
          // Download and install eksctl
          sh 'curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp'
          sh 'mv /tmp/eksctl /usr/local/bin'
          sh 'eksctl version'
          
          // Download and install kubectl
          sh 'curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"'
          sh 'chmod +x kubectl'
          sh 'mv kubectl /usr/local/bin/'
          
          // Create EKS cluster
          sh 'eksctl create cluster --name demo-ekscluster --region ap-south-1 --version 1.31 --nodegroup-name linux-nodes --node-type t2.micro --nodes 2'
        }
      }
    }

    stage("Clone code from GitHub") {
      steps {
        script {
          checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'GITHUB_CREDENTIALS', url: 'https://github.com/honeypvt/Deploy-NodeApp-to-AWS-EKS-using-Jenkins-Pipeline']])
        }
      }
    }
     
    stage('Node JS Build') {
      steps {
        sh 'npm install'
      }
    }
  
    stage('Build Node JS Docker Image') {
      steps {
        script {
          sh 'docker build -t kushakumar/node-app-1.0 .'
        }
      }
    }

    stage('Deploy Docker Image to DockerHub') {
      steps {
        script {
          withCredentials([string(credentialsId: 'devopshintdocker', variable: 'devopshintdocker')]) {
            sh 'docker login -u kushakumar -p ${devopshintdocker}'
          }
          sh 'docker push kushakumar/node-app-1.0'
        }
      }   
    }
         
    stage('Deploying Node App to Kubernetes') {
      steps {
        script {
          sh ('aws eks update-kubeconfig --name demo-ekscluster --region ap-south-1')
          sh "kubectl get ns"
          sh "kubectl apply -f nodejsapp.yaml"
        }
      }
    }
  }
}
