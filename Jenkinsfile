pipeline {
    agent { label 'ec2-node' }

    stages {
        stage('Agent Test') {
            steps {
                sh '''
                whoami
                hostname
                docker --version
                java -version
                '''
            }
        }
    }
}
