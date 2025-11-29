pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'your-dockerhub-username/spring-petclinic'
        SONARQUBE_ENV = 'sonarqube'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Krishnam0303/spring-petclinic.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv(SONARQUBE_ENV) {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:latest ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh "docker push ${DOCKER_IMAGE}:latest"
                }
            }
        }

        stage('Deploy to App Server') {
            steps {
                sshagent (credentials: ['app-server-ssh']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ubuntu@<APP_SERVER_PUBLIC_IP> '
                      sudo docker pull ${DOCKER_IMAGE}:latest &&
                      sudo docker rm -f spring-petclinic || true &&
                      sudo docker run -d --name spring-petclinic -p 80:8080 ${DOCKER_IMAGE}:latest
                    '
                    """
                }
            }
        }
    }
}
