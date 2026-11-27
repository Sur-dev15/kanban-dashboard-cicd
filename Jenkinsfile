
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
                '''
            }
        }

        stage('Verify Container Health') {
            steps {
                sh '''
                    echo "Checking Docker container health..."

                    for i in $(seq 1 30); do
                        HEALTH=$(docker inspect \
                            --format='{{if .State.Health}}{{.State.Health.Status}}{{else}}no-healthcheck{{end}}' \
                            kanban-dashboard)

                        echo "Health status: $HEALTH"

                        if [ "$HEALTH" = "healthy" ]; then
                            echo "Container is healthy!"
                            break
                        fi

                        if [ "$HEALTH" = "unhealthy" ] || [ "$HEALTH" = "no-healthcheck" ]; then
                            echo "Container health verification failed!"
                            docker logs kanban-dashboard
                            exit 1
                        fi

                        sleep 2
                    done

                    if [ "$HEALTH" != "healthy" ]; then
                        echo "Container did not become healthy within 60 seconds."
                        docker logs kanban-dashboard
                        exit 1
                    fi

                    echo "Checking application HTTP response..."

                    curl -f --retry 5 --retry-delay 2 \
                        --retry-connrefused http://localhost/

                    echo "Application validation successful!"
                '''
            }
        }
    }
}
