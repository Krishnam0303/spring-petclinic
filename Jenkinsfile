pipeline {
  agent any

  environment {
    DOCKER_IMAGE = "spring-petclinic:latest"
    DOCKER_SERVER = "54.91.158.155"
  }

  stages {

    stage('Build with Maven') {
      steps {
        sh '''
          mvn clean package \
          -DskipTests \
          -Dcyclonedx.skip=true
        '''
      }
    }

    stage('Upload Artifact to Nexus') {
      steps {
        sh '''
          mvn deploy \
          -DskipTests \
          -Dcyclonedx.skip=true
        '''
      }
    }

    stage('Docker Build') {
      steps {
        sh '''
          docker build -t ${DOCKER_IMAGE} .
        '''
      }
    }

    stage('Deploy to Docker EC2') {
      steps {
        sh '''
          ssh -o StrictHostKeyChecking=no ubuntu@${DOCKER_SERVER} "
            docker stop petclinic || true &&
            docker rm petclinic || true &&
            docker run -d --name petclinic -p 8080:8080 ${DOCKER_IMAGE}
          "
        '''
      }
    }
  }

  post {
    success {
      echo "CI/CD Pipeline completed successfully"
    }
    failure {
      echo "CI/CD Pipeline failed. Check logs."
    }
  }
}
