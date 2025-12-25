pipeline {
  agent any

  environment {
    IMAGE = "diarkin/di-webapp"
    DOCKER_CREDS = "dockerhub-di"
  }

  stages {

    stage("Checkout") {
      steps {
        checkout scm
      }
    }

    stage("Build Docker Image") {
      steps {
        sh "docker build -t di-webapp:build ."
      }
    }

    stage("Tag & Push to Docker Hub") {
      steps {
        script {
          env.TAG = "di-${BUILD_NUMBER}"
        }
        withCredentials([
          usernamePassword(
            credentialsId: DOCKER_CREDS,
            usernameVariable: 'U',
            passwordVariable: 'P'
          )
        ]) {
          sh """
            docker tag di-webapp:build ${IMAGE}:${TAG}
            echo "${P}" | docker login -u "${U}" --password-stdin
            docker push ${IMAGE}:${TAG}
          """
        }
      }
    }

    stage("Deploy to Kubernetes") {
      steps {
        sh """
          sed -i 's|di-PLACEHOLDER|di-${BUILD_NUMBER}|g' k8s/deployment-di.yaml
          kubectl apply -f k8s/
          kubectl rollout status deployment/di-webapp --timeout=180s
        """
      }
    }
  }
}
