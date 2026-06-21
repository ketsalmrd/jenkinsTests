pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/ketsalmrd/jenkinsTests.git'
            }
        }

        stage('Build') {
            steps {
                sh 'echo "Building..."'
            }
        }

        stage('Test') {
            steps {
                sh 'echo "Running tests..."'
            }
        }
    }
}