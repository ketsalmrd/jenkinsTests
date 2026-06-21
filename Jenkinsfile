pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/tu-repo.git'
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