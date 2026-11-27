
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

        stage('Save Previous Deployment') {
            steps {
                sh '''
                    if docker inspect kanban-dashboard >/dev/null 2>&1; then
                        docker inspect --format='{{.Image}}' \
                            kanban-dashboard > "$WORKSPACE/previous-image-$BUILD_NUMBER.txt"
                        echo "Previous deployment image saved."
                    else
                        echo "NONE" > "$WORKSPACE/previous-image-$BUILD_NUMBER.txt"
                        echo "No previous deployment exists."
                    fi
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

                    HEALTH="starting"

                    for i in $(seq 1 30); do
                        HEALTH=$(docker inspect \
                            --format='{{if .State.Health}}{{.State.Health.Status}}{{else}}no-healthcheck{{end}}' \
                            kanban-dashboard)

                        echo "Health status: $HEALTH"

                        if [ "$HEALTH" = "healthy" ]; then
                            break
                        fi

                        if [ "$HEALTH" = "unhealthy" ] || [ "$HEALTH" = "no-healthcheck" ]; then
                            docker logs kanban-dashboard
                            exit 1
                        fi

                        sleep 2
                    done

                    if [ "$HEALTH" != "healthy" ]; then
                        docker logs kanban-dashboard
                        exit 1
                    fi

                    curl -f --retry 5 --retry-delay 2 \
                        --retry-connrefused http://localhost/

                    echo "Application validation successful!"
                '''
            }
        }
    }

    post {
        failure {
            echo 'Deployment failed. Starting automatic rollback...'

            sh '''
                PREVIOUS_FILE="$WORKSPACE/previous-image-$BUILD_NUMBER.txt"

                if [ ! -f "$PREVIOUS_FILE" ]; then
                    echo "No saved deployment found. Nothing to roll back."
                    exit 0
                fi

                PREVIOUS_IMAGE=$(cat "$PREVIOUS_FILE")

                docker stop kanban-dashboard || true
                docker rm kanban-dashboard || true

                if [ "$PREVIOUS_IMAGE" = "NONE" ]; then
                    echo "No previous version was available."
                    exit 0
                fi

                echo "Restoring previous image: $PREVIOUS_IMAGE"

                docker run -d \
                    --name kanban-dashboard \
                    --restart unless-stopped \
                    --memory=512m \
                    --cpus=1 \
                    -p 80:8080 \
                    "$PREVIOUS_IMAGE"

                sleep 5

                curl -f --retry 10 --retry-delay 2 \
                    --retry-connrefused http://localhost/

                echo "Previous deployment restored successfully."
            '''
        }
    }
}
