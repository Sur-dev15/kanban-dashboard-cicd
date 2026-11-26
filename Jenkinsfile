

pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'guvisurangma/kanban-dashboard'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Sur-dev15/kanban-dashboard-cicd.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t $DOCKER_IMAGE:$BUILD_NUMBER .
                    docker tag $DOCKER_IMAGE:$BUILD_NUMBER $DOCKER_IMAGE:latest
                '''
            }
        }

        stage('Docker Hub Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login \
                        -u "$DOCKER_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                sh '''
                    docker push $DOCKER_IMAGE:$BUILD_NUMBER
                    docker push $DOCKER_IMAGE:latest
                '''
            }
        }

        stage('Deploy on EC2') {
            steps {
                sh '''
                    docker stop kanban-dashboard || true
                    docker rm kanban-dashboard || true

                    docker run -d \
                        --name kanban-dashboard \
                        --restart unless-stopped \
                        --memory=512m \
                        --cpus=1 \
                        -p 80:8080 \
                        $DOCKER_IMAGE:$BUILD_NUMBER

                    sleep 5
                    curl -f http://localhost/
                '''
            }
        }
    }
}
