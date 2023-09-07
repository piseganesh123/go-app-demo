pipeline {

  environment {
    PROJECT = "REPLACE_WITH_YOUR_PROJECT_ID"
    APP_NAME = "gceme"
    FE_SVC_NAME = "${APP_NAME}-frontend"
    CLUSTER = "jenkins-cd"
    CLUSTER_ZONE = "us-east1-d"
    IMAGE_TAG = "gcr.io/${PROJECT}/${APP_NAME}:${env.BRANCH_NAME}.${env.BUILD_NUMBER}"
    JENKINS_CRED = "${PROJECT}"
  }

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
//    image: gcr.io/cloud-builders/kubectl@sha256:41cdfdf5e4963c3f437a9e405d71489f4e05b26b1cdf5dc9cf33d5afc48d5370
//image: bitnami/kubectl:1.27.5
}
  }
  stages {
    stage('Deploy Canary') {
      // Canary branch
      when { branch 'canary' }
      steps {
        container('kubectl') {
          // Change deployed image in canary to the one we just built
          sh("sed -i.bak 's#corelab/gceme:1.0.0#${IMAGE_TAG}#' ./k8s/canary/*.yaml")
//        step([$class: 'KubernetesEngineBuilder', namespace:'production', projectId: env.PROJECT, clusterName: env.CLUSTER, zone: env.CLUSTER_ZONE, manifestPattern: 'k8s/services', credentialsId: env.JENKINS_CRED, verifyDeployments: false])
//         step([$class: 'KubernetesEngineBuilder', namespace:'production', projectId: env.PROJECT, clusterName: env.CLUSTER, zone: env.CLUSTER_ZONE, manifestPattern: 'k8s/canary', credentialsId: env.JENKINS_CRED, verifyDeployments: true])
          sh("echo http://`kubectl --namespace=production get service/${FE_SVC_NAME} -o jsonpath='{.status.loadBalancer.ingress[0].ip}'` > ${FE_SVC_NAME}")
          sh("echo In Deploy canary stage")
          sh("kubectl get pods")
        }
      }
    }
    stage('Deploy Production') {
      // Production branch
      when { branch 'main' }
      steps{
        container('kubectl') {
        // Change deployed image in canary to the one we just built
//          sh("sed -i.bak 's#corelab/gceme:1.0.0#${IMAGE_TAG}#' ./k8s/production/*.yaml")
//          sh("echo http://`kubectl --namespace=production get service/${FE_SVC_NAME} -o jsonpath='{.status.loadBalancer.ingress[0].ip}'` > ${FE_SVC_NAME}")
          sh("echo In Deploy Production stage")
          sh("pwd")
          sh("kubectl apply -f ./sample-app/k8s/production/app-deployment.yaml")
          sh("kubectl apply -f ./sample-app/k8s/production/java-demo-service.yaml")
          
        }
      }
    }
    stage('Deploy Dev') {
      // Developer Branches
      when {
        not { branch 'main' }
        not { branch 'canary' }
      }
      steps {
        container('kubectl') {
          // Create namespace if it doesn't exist
          sh("kubectl get ns ${env.BRANCH_NAME} || kubectl create ns ${env.BRANCH_NAME}")
          // Don't use public load balancing for development branches
          sh("echo In Deploy Dev stage")
//          sh("sed -i.bak 's#LoadBalancer#ClusterIP#' ./k8s/services/frontend.yaml")
//          sh("sed -i.bak 's#corelab/gceme:1.0.0#${IMAGE_TAG}#' ./k8s/dev/*.yaml")
          echo 'To access your environment run `kubectl proxy`'
          echo "Then access your service via http://localhost:8001/api/v1/proxy/namespaces/${env.BRANCH_NAME}/services/${FE_SVC_NAME}:80/"
        }
      }
    }
  }
}
