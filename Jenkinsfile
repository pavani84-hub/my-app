pipeline {
  agent any
  environment {
    IMAGE = "https://github.com/pavani84-hub/my-app:latest"
    KUBECONFIG_CRED = 'kubeconfig-credentials'
    DOCKERHUB_CRED = 'dockerhub-credentials'
  }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Build & Push Docker') {
      steps {
        script {
          docker.withRegistry('', DOCKERHUB_CRED) {
            def img = docker.build(IMAGE)
            img.push()
          }
        }
      }
    }
    stage('Deploy to Kubernetes') {
      steps {
        withCredentials([file(credentialsId: KUBECONFIG_CRED, variable: 'KUBECONFIG')]) {
          sh '''
            mkdir -p ~/.kube
            cp "$KUBECONFIG" ~/.kube/config
            kubectl set image deployment/my-app my-app=$IMAGE --record
            kubectl rollout status deployment/my-app
          '''
        }
      }
    }
  }
  post {
    success { echo 'Deployment to Kubernetes successful!' }
    failure { echo 'Pipeline failed — check logs.' }
  }
}
