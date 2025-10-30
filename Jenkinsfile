pipeline {
  agent any

  environment {
    IMAGE = "charan160704/mywebapp"
    IMAGE_TAG = "${env.BUILD_NUMBER}"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build app') {
      steps {
        // Node: install & build (adapt to your tech stack)
        sh 'npm ci'
        sh 'npm run build || true'   // optional
      }
    }

    stage('Build docker image') {
      steps {
        sh "docker build -t ${IMAGE}:${IMAGE_TAG} ."
      }
    }

    stage('Login to DockerHub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
        }
      }
    }

    stage('Push image') {
      steps {
        sh "docker push ${IMAGE}:${IMAGE_TAG}"
        // optional: update 'latest' tag
        sh "docker tag ${IMAGE}:${IMAGE_TAG} ${IMAGE}:latest || true"
        sh "docker push ${IMAGE}:latest || true"
      }
    }

    stage('Deploy') {
      steps {
        // On the target host you could use docker run, or Docker Compose, or kubectl.
        // If Jenkins manages the target host, you can run:
        sh '''
          docker rm -f mywebapp || true
          docker run -d --name mywebapp -p 3000:3000 ${IMAGE}:${IMAGE_TAG}
        '''
      }
    }
  }

  post {
    always {
      echo "Pipeline finished"
    }
  }
}

