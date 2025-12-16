pipeline {
  agent any

  stages {

    stage('Build with Maven') {
      steps {
        sh 'mvn clean package -DskipTests'
      }
    }

    stage('Upload Artifact to Nexus') {
      steps {
        sh 'mvn deploy -DskipTests'
      }
    }

    stage('Docker Build') {
      steps {
        sh 'docker build -t spring-petclinic:latest .'
      }
    }

    stage('Deploy to Docker EC2') {
      steps {
        sh '''
        ssh ubuntu@54.91.158.155 "
          docker stop petclinic || true &&
          docker rm petclinic || true &&
          docker run -d --name petclinic -p 8080:8080 spring-petclinic:latest
        "
        '''
      }
    }
  }
}
