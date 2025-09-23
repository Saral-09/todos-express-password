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
                scannerHome = tool 'sonarqube-tool'
            }
            steps {
                withSonarQubeEnv('sonarqube-installation') {
                    sh """
                        ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=myproject \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=$SONAR_HOST_URL \
                        -Dsonar.login=$SONAR_AUTH_TOKEN
                    """
                }
            }
        }
        
        stage('Security Scan') {
            steps {
                echo "Running Trivy scan on Docker image..."
                sh """
                    touch trivy-report.txt
                    trivy image --scanners vuln --exit-code 0 \
                    --severity HIGH,CRITICAL \
                    -o trivy-report.txt ${env.DOCKER_IMAGE}:latest || true
                """
                emailext(
                    to: 'saralbajimaya09@gmail.com',
                    subject: "Trivy Scan Report for Build #${env.BUILD_NUMBER}",
                    body: """\
Hello,

Please find the Trivy container security scan report attached for Build #${env.BUILD_NUMBER}.

Regards,  
Jenkins
                    """,
                    attachmentsPattern: 'trivy-report.txt'
                )
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

    stage('Deploy') {
        steps {
                echo "Deploying Docker image..."
                script {
                    sh """
                    docker stop todos-app || true
                    docker rm todos-app || true
                    docker run -d --name todos-app -p 3000:3000 ${env.DOCKER_IMAGE}:latest
                    """
                }
            }
        }
        
    stage('Release') {
        steps {
            script {
                echo "Creating Git tag for release..."
                sh """
                    git config user.name "Jenkins"
                    git config user.email "jenkins@example.com"
                    git tag -a "v${BUILD_NUMBER}" -m "Release build ${BUILD_NUMBER}"
                    git push origin "v${BUILD_NUMBER}"
                """
                
                echo "Tagging Docker image for release..."
                sh """
                    docker tag ${DOCKER_IMAGE}:latest ${DOCKER_IMAGE}:v${BUILD_NUMBER}
                    docker push ${DOCKER_IMAGE}:v${BUILD_NUMBER}
                """
                
                echo "Release v${BUILD_NUMBER} completed successfully!"
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
