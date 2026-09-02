
pipeline {

    agent any

    environment {

        // Docker Hub image
        DOCKER_IMAGE = "samzcode/jenkins"

    }

    stages {

        // ==========================================
        // 1. CHECKOUT
        // ==========================================

        stage('Checkout') {

            steps {

                echo 'Checking out source code...'

                checkout scm
            }
        }


        // ==========================================
        // 2. TEST
        // ==========================================

        stage('Test') {

            steps {

                echo 'Running application tests...'

                // For a simple HTML/CSS/JS website,
                // we verify that the website files exist.

                sh '''
                    test -f web/Dockerfile
                    test -f web/index.html

                    echo "Required files found."
                '''
            }
        }

             // ==========================================
        // 4. LOGIN TO DOCKER HUB
        // ==========================================

        stage('Docker Login') {

            steps {

                echo 'Logging into Docker Hub...'

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {

                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                        --username "$DOCKER_USERNAME" \
                        --password-stdin
                    '''
                }
            }
        }




        // ==========================================
        // 3. BUILD DOCKER IMAGE
        // ==========================================

        stage('Docker Build') {

            steps {

                echo 'Building Docker image...'

                sh '''
                    docker build \
                    -t ${DOCKER_IMAGE}:latest \
                    ./web
                '''
            }
        }
        
        // ==========================================
        // 5. PUSH IMAGE TO DOCKER HUB
        // ==========================================

        stage('Push to Docker Hub') {

            steps {

                echo 'Pushing Docker image to Docker Hub...'

                sh '''
                    docker push ${DOCKER_IMAGE}:latest
                '''
            }
        }

        stage('Test SSH') {
            steps {
                sshagent(['aws-ec2-ssh']) {
                    sh '''
                        echo "Keys loaded into SSH agent:"
                        ssh-add -l
        
                        echo "Testing SSH..."
                        ssh -vvv -o StrictHostKeyChecking=no ubuntu@16.16.149.119 "whoami"
                    '''
                }
            }
        }


        // ==========================================
        // 6. DEPLOY TO AWS EC2
        // ==========================================

        stage('Deploy to AWS') {

            steps {

                echo 'Deploying application to AWS EC2...'

                sshagent(['aws-ec2-ssh']) {

                    sh '''

                        ssh -o StrictHostKeyChecking=no \
                        ubuntu@16.16.149.119 << 'EOF'

                        set -e

                        echo "Starting deployment..."

                        # Create deployment directory
                        mkdir -p ~/my-site

                        cd ~/my-site

                        # Create docker-compose.yml
                        cat > docker-compose.yml << COMPOSE
                        services:
                          website:
                            image: ${DOCKER_IMAGE}:latest
                            container_name: my-website
                            ports:
                              - "80:80"
                            restart: unless-stopped
                        COMPOSE

                        # Pull latest Docker image
                        echo "Pulling latest image..."

                        docker pull ${DOCKER_IMAGE}:latest

                        # Stop old container
                        echo "Stopping old container..."

                        docker compose down || true

                        # Start new container
                        echo "Starting new container..."

                        docker compose up -d

                        # Remove unused images
                        docker image prune -f

                        # Show running containers
                        echo "Running containers:"

                        docker ps

                        echo "Deployment completed successfully!"

                        EOF
                    '''
                }
            }
        }
    }


    // ==========================================
    // PIPELINE RESULT
    // ==========================================

    post {

        success {

            echo '====================================='
            echo 'CI/CD PIPELINE COMPLETED SUCCESSFULLY'
            echo '====================================='
        }

        failure {

            echo '====================================='
            echo 'CI/CD PIPELINE FAILED'
            echo '====================================='
        }
    }
}
