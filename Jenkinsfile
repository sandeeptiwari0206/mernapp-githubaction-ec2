pipeline {
    agent any

    environment {
        FRONTEND_IMAGE = "sandeeptiwari0206/mern-frontend"
        BACKEND_IMAGE  = "sandeeptiwari0206/mern-backend"
        DOCKER_CREDS   = "dockerhub-creds"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/sandeeptiwari0206/mernapp.git'
            }
        }

        stage('Build Frontend Image') {
            steps {
                dir('frontend') {
                    bat 'docker build -t %FRONTEND_IMAGE% .'
                }
            }
        }

        stage('Build Backend Image') {
            steps {
                // Dockerfile is at root
                bat 'docker build -t %BACKEND_IMAGE% .'
            }
        }

        stage('Docker Hub Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: DOCKER_CREDS,
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat 'echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin'
                }
            }
        }

        stage('Push Images to Docker Hub') {
            steps {
                bat """
                  docker push %FRONTEND_IMAGE%
                  docker push %BACKEND_IMAGE%
                """
            }
        }
    }

    post {
        success {
            echo "✅ CI Pipeline successful: Images built & pushed to Docker Hub!"
        }
        failure {
            echo "❌ CI Pipeline failed. Check Jenkins logs."
        }
    }
}
