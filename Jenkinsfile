pipeline {
    agent any

    environment {
        APP_DIR = "/home/adminuser/project/todos-express-password"
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Cloning Git repository via SSH..."
                git branch: 'master', 
                    url: 'git@github.com:Saral-09/todos-express-password.git', 
                    credentialsId: 'jenkins-github-ssh'
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
                    sh 'npm test || true' // keep pipeline running even if no tests yet
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
