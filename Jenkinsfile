
pipeline {
    agent any

    environment {
        IMAGE_NAME = 'manasa2514/cicd'
    }

    stages {

        // Stage 1: Checkout source code from GitHub
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        // Stage 2: Install Python dependencies
        stage('Install Dependencies') {
            steps {
                sh '''
                    python3 -m venv .venv
                    .venv/bin/pip install -r requirements.txt
                '''
            }
        }

        // Stage 3: Run unit tests
        stage('Run Tests') {
            steps {
                sh '.venv/bin/pytest -v'
            }
        }

        // Stage 4: Build Docker image
        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t $IMAGE_NAME:$BUILD_NUMBER .
                    docker tag $IMAGE_NAME:$BUILD_NUMBER $IMAGE_NAME:latest
                '''
            }
        }

        // Stage 5: Push Docker image to Docker Hub
        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login \
                            -u "$DOCKER_USER" --password-stdin

                        docker push $IMAGE_NAME:$BUILD_NUMBER
                        docker push $IMAGE_NAME:latest

                        docker logout
                    '''
                }
            }
        }
    }
    // Stage 6: Deploy application using Ansible
        stage('Deploy with Ansible') {
            steps {
                sh '''
                    ansible-playbook -i ansible/inventory ansible/deploy.yml
                '''
            }
        }
    }
    post {
        success {
            echo 'CI pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Please check the Console Output.'
        }
    }
}

