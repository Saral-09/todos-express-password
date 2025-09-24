pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "saral09/todos-express-password"
        KALI_VM = "kali@192.168.1.233"
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
/*
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
                    trivy image --scanners vuln --exit-code 0 \
                    --severity HIGH,CRITICAL \
                    --timeout 15m \
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
*/
        stage('Deploy Stage') {
            steps {
                script {
                    echo "Building Docker image..."
                    sh """
                        docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .
                        docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest
                    """

                    echo "Deploying Docker container on Jenkins server (staging)..."
                    sh """
                        if [ \$(docker ps -aq -f name=todos-app) ]; then
                            docker stop todos-app
                            docker rm todos-app
                        fi
                        docker run -d --name todos-app -p 3000:3000 ${DOCKER_IMAGE}:latest
                    """

                    echo "Pushing Docker image to Docker Hub..."
                    withCredentials([usernamePassword(credentialsId: 'DockerHub', 
                                                     usernameVariable: 'DOCKER_USER', 
                                                     passwordVariable: 'DOCKER_PASS')]) {
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                        sh "docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}"
                        sh "docker push ${DOCKER_IMAGE}:latest"
                    }
                }
            }
        }

        stage('Release (Production on Kali VM)') {
            steps {
                echo "Deploying Docker container on Kali VM (production)..."
                sh """
                ssh ${KALI_VM} '
                    sudo docker pull ${DOCKER_IMAGE}:latest
                    if [ \$(sudo docker ps -aq -f name=todos-app) ]; then
                        sudo docker stop todos-app
                        sudo docker rm todos-app
                    fi
                    sudo docker run -d --name todos-app -p 3000:3000 ${DOCKER_IMAGE}:latest
                '
                """
            }
        }

        stage('Git Tag Release') {
            steps {
                echo "Creating Git tag for release..."
                sh """
                    git config user.name "Jenkins"
                    git config user.email "jenkins@example.com"
                    git tag -a "v${BUILD_NUMBER}" -m "Release build ${BUILD_NUMBER}"
                    git push origin "v${BUILD_NUMBER}"
                """
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
