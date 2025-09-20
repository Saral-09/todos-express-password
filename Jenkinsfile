pipeline {
    agent any

    environment {
        APP_DIR = "/home/adminuser/project/todos-express-password"
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Cloning Git repository..."
                git branch: 'main', url: 'https://github.com/YOUR_USERNAME/todos-express-password.git'
            }
        }

        stage('Build') {
            steps {
                echo "Installing dependencies..."
                dir("${APP_DIR}") {
                    sh 'npm install'
                }
            }
        }

        stage('Test') {
            steps {
                echo "Running automated tests..."
                dir("${APP_DIR}") {
                    sh 'npm test'
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed. Check the logs for errors."
        }
    }
}
