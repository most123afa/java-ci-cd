pipeline {
  agent any

  environment {
    DOCKER_IMAGE = "alyyoussef/java-app"
    TAG = "${BUILD_NUMBER}"
  }

  stages {

    stage('Checkout') {
      steps {
        git credentialsId: 'git-creds', url: 'https://github.com/you/java-app.git'
      }
    }

    stage('Build & Test') {
      steps {
        sh 'mvn clean verify'
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv('sonarqube') {
          sh 'mvn sonar:sonar'
        }
      }
    }

    stage('Quality Gate') {
      steps {
        timeout(time: 1, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
      }
    }

    stage('Docker Build & Push') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'dockerhub-creds',
          usernameVariable: 'USER',
          passwordVariable: 'PASS'
        )]) {
          sh """
            docker login -u $USER -p $PASS
            docker build -t $DOCKER_IMAGE:$TAG .
            docker push $DOCKER_IMAGE:$TAG
          """
        }
      }
    }

    stage('Update Helm Values') {
      steps {
        sh """
          git clone https://github.com/you/java-app-manifests.git
          cd java-app-manifests
          sed -i 's/tag:.*/tag: ${TAG}/' helm/java-app/values.yaml
          git commit -am "Update image tag to ${TAG}"
          git push
        """
      }
    }
  }
}
