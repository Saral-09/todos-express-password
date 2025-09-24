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

                emailext(
                    to: 'saralbajimaya09@gmail.com',
                    subject: "Build Completed - Build #${env.BUILD_NUMBER}",
                    body: """\
Hello,

Build stage for Build #${env.BUILD_NUMBER} completed successfully.  
Please find attached the package-lock.json as the build artefact.

Regards,  
Jenkins
                    """,
                    attachmentsPattern: 'package-lock.json'
                )
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                sh 'npm test'
        
                // Send test results email with actual report file
                emailext(
                    to: 'saralbajimaya09@gmail.com',
                    subject: "Test Stage Completed - Build #${env.BUILD_NUMBER}",
                    body: """\
Hello,

Test stage for Build #${env.BUILD_NUMBER} completed successfully.  
Please find attached the test results.

Regards,  
Jenkins
        """,
                    attachmentsPattern: 'test-reports/results.xml'
                )
            }
        }

        stage('Security Scan') {
            steps {
                echo "Running Trivy scan on Docker image..."
                sh """
                    touch trivy-report.txt
                    trivy image --scanners vuln --exit-code 0 \
                    --severity HIGH,CRITICAL \
                    --timeout 15m \
                    -o trivy-report.txt ${env.DOCKER_IMAGE}:latest || true
                """
                emailext(
                    to: 'saralbajimaya09@gmail.com',
                    subject: "Trivy Security Scan Report - Build #${env.BUILD_NUMBER}",
                    body: """\
Hello,

Security scan for Build #${env.BUILD_NUMBER} completed successfully.  
Please find attached the Trivy scan report.

Regards,  
Jenkins
                    """,
                    attachmentsPattern: 'trivy-report.txt'
                )
            }
        }

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
                    emailext(
                        to: 'saralbajimaya09@gmail.com',
                        subject: "Staging Deploy Completed - Build #${env.BUILD_NUMBER}",
                        body: """\
Hello,

Staging deployment for Build #${env.BUILD_NUMBER} completed successfully.

Regards,  
Jenkins
                        """
                    )
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
                emailext(
                    to: 'saralbajimaya09@gmail.com',
                    subject: "Production Release Completed - Build #${env.BUILD_NUMBER}",
                    body: """\
Hello,

Production release for Build #${env.BUILD_NUMBER} has been successfully deployed on the production VM.

Regards,  
Jenkins
                    """
                )
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
                emailext(
                    to: 'saralbajimaya09@gmail.com',
                    subject: "Git Tag Created - Build #${env.BUILD_NUMBER}",
                    body: """\
Hello,

Git tag v${BUILD_NUMBER} has been created for Build #${env.BUILD_NUMBER}.

Regards,  
Jenkins
                    """
                )
            }
        }

        stage('Monitoring and Alerting Stage') {
            steps {
                script {
                    echo "Checking Datadog agent on remote VM..."
                    sh """
                        ssh ${KALI_VM} 'sudo datadog-agent status | grep "Docker"'
                    """
                    emailext(
                        to: 'saralbajimaya09@gmail.com',
                        subject: "Monitoring Check Completed - Build #${env.BUILD_NUMBER}",
                        body: """\
Hello,

Monitoring check for Build #${env.BUILD_NUMBER} has been completed.  
Datadog agent is running and reporting Docker metrics.

Regards,  
Jenkins
                        """
                    )
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
