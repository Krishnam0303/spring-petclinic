pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'krishnam030303/spring-petclinic'
        BRANCH_NAME = 'main'
    }

    tools {
        jdk 'jdk17'
        maven 'maven'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: BRANCH_NAME, url: 'https://github.com/Krishnam0303/spring-petclinic.git'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:latest ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                                  usernameVariable: 'DOCKER_USER',
                                                  passwordVariable: 'DOCKER_PASS')]) {

                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh "docker push ${DOCKER_IMAGE}:latest"
                }
            }
        }

        stage('Deploy to App Server') {
            steps {
                sshagent (credentials: ['app-server-ssh']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ubuntu@65.2.4.162 '
                        sudo docker pull ${DOCKER_IMAGE}:latest &&
                        sudo docker rm -f spring-petclinic || true &&
                        sudo docker run -d --name spring-petclinic -p 80:8080 ${DOCKER_IMAGE}:latest
                    '
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Deployment Completed Successfully!"
        }
        failure {
            echo "Pipeline Failed — Check logs."
        }
    }
}
