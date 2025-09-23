pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "saral09/todos-express-password"
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Cloning Git repository..."
                git branch: 'master', 
                    url: 'git@github.com:Saral-09/todos-express-password.git', 
                    credentialsId: '66a0ea00-f0ac-46cd-888f-ce1a49ec1121'
            }
        }

        stage('Build') {
            steps {
                echo 'Installing dependencies...'
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                sh 'npm test'
            }
        }

        stage('Code Quality Analysis') {
            environment {
                scannerHome = tool 'qube' // Must match your SonarQube Scanner installation name
            }
            steps {
                withSonarQubeEnv('SERVER-SONAR') { // Must match your SonarQube server config
                    withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_TOKEN')]) {
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=todos-express-password -Dsonar.sources=. -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.login=$SONAR_TOKEN"
                    }
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    echo "Building Docker image..."
                    sh "docker build -t ${env.DOCKER_IMAGE}:${BUILD_NUMBER} ."
                    sh "docker tag ${env.DOCKER_IMAGE}:${BUILD_NUMBER} ${env.DOCKER_IMAGE}:latest"

                    echo "Pushing Docker image to Docker Hub..."
                    withCredentials([usernamePassword(credentialsId: 'DockerHub', 
                                                     usernameVariable: 'DOCKER_USER', 
                                                     passwordVariable: 'DOCKER_PASS')]) {
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                        sh "docker push ${env.DOCKER_IMAGE}:${BUILD_NUMBER}"
                        sh "docker push ${env.DOCKER_IMAGE}:latest"
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed. Check logs for details."
        }
    }
}
