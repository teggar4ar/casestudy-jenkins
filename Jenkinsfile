pipeline {
  agent any
  environment {
    IMAGE = "azeshion21/demo-app"
    TAG = "latest"
    DOCKER_CRED = "dockerhub-cred"
    KUBECONFIG_CRED = "kubeconfig-dev"
    NAMESPACE = "default"
    HELM_RELEASE = "casestudy-jenkins1"
  }
  stages {
    stage('Checkout') {
      steps {
        git url: 'https://github.com/orion2182/casestudy-jenkins.git', branch: 'main'
      }
    }
    stage('Build Docker Image') {
      steps {
        script {
          docker.build("${IMAGE}:${TAG}")
        }
      }
    }
    stage('Push Docker Image') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${DOCKER_CRED}", usernameVariable: 'USER', passwordVariable: 'PASS')]) {
          sh """
            echo "$PASS" | docker login -u "$USER" --password-stdin
            docker push ${IMAGE}:${TAG}
          """
        }
      }
    }
    stage('Deploy to Kubernetes') {
      steps {
        withCredentials([file(credentialsId: "${KUBECONFIG_CRED}", variable: 'KUBE_FILE')]) {
          sh '''
            export KUBECONFIG=$KUBE_FILE
            helm upgrade --install $HELM_RELEASE ./helm \
              --set image.repository=$IMAGE \
              --set image.tag=$TAG \
              --namespace $NAMESPACE --create-namespace
          '''
        }
      }
    }
  }
}
