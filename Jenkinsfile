pipeline {
  agent {
    kubernetes {
      label 'java-app-agent'
      defaultContainer 'jnlp'
      yaml """
apiVersion: v1
kind: Pod
metadata:
labels:
  component: ci
spec:
  serviceAccountName: cd-jenkins
  containers:
  - name: kubectl
    image: gcr.io/cloud-builders/kubectl@sha256:41cdfdf5e4963c3f437a9e405d71489f4e05b26b1cdf5dc9cf33d5afc48d5370
    
    command:
    - cat
    tty: true
"""
}
  }
  stages {
    stage('Deploy Production') {
      // Production branch
      when { branch 'main' }
      steps{
        container('kubectl') {
          sh("echo In Deploy Production stage")
          sh("pwd")
          sh("kubectl apply -f ./sample-app/k8s/production/app-deployment.yaml")
          sh("kubectl apply -f ./sample-app/k8s/production/java-demo-service.yaml")
        }
      }
    }
  }
}
