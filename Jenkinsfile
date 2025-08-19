pipeline {
  agent any
  environment {
    IMAGE_NAME = "demo-ci-cd:latest"
  }
  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }
    stage('Build & Test') {
      steps {
        sh 'mvn -B clean package'
      }
    }
    stage('Build Docker Image') {
      steps {
        sh 'docker build -t $IMAGE_NAME .'
      }
    }
    stage('Push Image (GHCR)') {
  steps {
    script {
      sh """
        docker tag demo-ci-cd:latest ghcr.io/alejandrapiz/demo-ci-cd:${BUILD_NUMBER}
        docker tag demo-ci-cd:latest ghcr.io/alejandrapiz/demo-ci-cd:latest
      """
    }
    withCredentials([usernamePassword(credentialsId: 'docker-registry-creds',
      usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
      sh '''
        echo "$DOCKER_PASS" | docker login ghcr.io -u "$DOCKER_USER" --password-stdin
        docker push ghcr.io/alejandrapiz/demo-ci-cd:${BUILD_NUMBER}
        docker push ghcr.io/alejandrapiz/demo-ci-cd:latest
      '''
    }
  }
}

    stage('Run Container') {
      steps {
        sh 'docker rm -f demo-ci-cd || true'
        sh 'docker run -d --name demo-ci-cd -p 8080:8080 $IMAGE_NAME'
      }
    }
  }
  post {
    always {
      junit '**/target/surefire-reports/*.xml'
      archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
    }
  }
}
