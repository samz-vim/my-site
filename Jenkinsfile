pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "YOUR_DOCKER_USERNAME/my-website"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'

                // If your project uses npm:
                sh 'npm ci'
                sh 'npm test'
            }
        }

        stage('Docker Build') {
            steps {
                sh """
                    docker build \
                    -t ${DOCKER_IMAGE}:latest \
                    ./web
                """
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                        -u "$DOCKER_USERNAME" \
                        --password-stdin
                    '''
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                sh """
                    docker push ${DOCKER_IMAGE}:latest
                """
            }
        }

        stage('Deploy to AWS') {
            steps {
                sshagent(['aws-ec2-ssh']) {

                    sh """
                        ssh -o StrictHostKeyChecking=no \
                        ec2-user@YOUR_EC2_IP '
                            
                            mkdir -p ~/my-website

                            cd ~/my-website

                            cat > docker-compose.yml << EOF
                            services:
                              website:
                                image: ${DOCKER_IMAGE}:latest
                                container_name: my-website
                                ports:
                                  - "80:80"
                                restart: unless-stopped
                            EOF

                            docker pull ${DOCKER_IMAGE}:latest

                            docker compose down || true

                            docker compose up -d

                            docker image prune -f

                            docker ps
                        '
                    """
                }
            }
        }
    }
}
