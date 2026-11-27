pipeline {
    agent any

    options {
        disableConcurrentBuilds()
    }

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

        stage('Identify Active Environment') {
            steps {
                sh '''
                    ACTIVE_PORT=$(grep -oE '127\\.0\\.0\\.1:808[12]' \
                        /etc/nginx/conf.d/kanban-upstream.conf \
                        | tail -n 1 | cut -d: -f2)

                    case "$ACTIVE_PORT" in
                        8081)
                            PREVIOUS_ENV="blue"
                            TARGET_ENV="green"
                            TARGET_PORT=8082
                            ;;
                        8082)
                            PREVIOUS_ENV="green"
                            TARGET_ENV="blue"
                            TARGET_PORT=8081
                            ;;
                        *)
                            echo "Unable to identify active environment."
                            exit 1
                            ;;
                    esac

                    echo "$PREVIOUS_ENV" > previous-env-$BUILD_NUMBER.txt
                    echo "$TARGET_ENV" > target-env-$BUILD_NUMBER.txt
                    echo "$TARGET_PORT" > target-port-$BUILD_NUMBER.txt

                    echo "Current environment: $PREVIOUS_ENV"
                    echo "New deployment environment: $TARGET_ENV"
                    echo "New deployment port: $TARGET_PORT"
                '''
            }
        }

        stage('Deploy to Inactive Environment') {
            steps {
                sh '''
                    TARGET_ENV=$(cat target-env-$BUILD_NUMBER.txt)
                    TARGET_PORT=$(cat target-port-$BUILD_NUMBER.txt)
                    TARGET_CONTAINER="kanban-$TARGET_ENV"

                    docker rm -f "$TARGET_CONTAINER" || true

                    docker run -d \
                        --name "$TARGET_CONTAINER" \
                        --restart unless-stopped \
                        --memory=512m \
                        --cpus=1 \
                        -p "$TARGET_PORT:8080" \
                        "$DOCKER_IMAGE:$BUILD_NUMBER"
                '''
            }
        }

        stage('Verify New Container Health') {
            steps {
                sh '''
                    TARGET_ENV=$(cat target-env-$BUILD_NUMBER.txt)
                    TARGET_PORT=$(cat target-port-$BUILD_NUMBER.txt)
                    TARGET_CONTAINER="kanban-$TARGET_ENV"
                    HEALTH="starting"

                    for i in $(seq 1 60); do
                        HEALTH=$(docker inspect \
                            --format='{{if .State.Health}}{{.State.Health.Status}}{{else}}no-healthcheck{{end}}' \
                            "$TARGET_CONTAINER")

                        echo "New container health: $HEALTH"

                        if [ "$HEALTH" = "healthy" ]; then
                            break
                        fi

                        if [ "$HEALTH" = "unhealthy" ] || \
                           [ "$HEALTH" = "no-healthcheck" ]; then
                            docker logs "$TARGET_CONTAINER"
                            exit 1
                        fi

                        sleep 2
                    done

                    if [ "$HEALTH" != "healthy" ]; then
                        docker logs "$TARGET_CONTAINER"
                        exit 1
                    fi

                    curl -f --retry 5 --retry-delay 2 \
                        --retry-connrefused "http://localhost:$TARGET_PORT/"

                    echo "New environment validation successful."
                '''
            }
        }

        stage('Switch Production Traffic') {
            steps {
                sh '''
                    TARGET_ENV=$(cat target-env-$BUILD_NUMBER.txt)

                    sudo -n /usr/local/sbin/kanban-switch "$TARGET_ENV"

                    echo "yes" > traffic-switched-$BUILD_NUMBER.txt

                    curl -f --retry 5 --retry-delay 2 \
                        --retry-connrefused http://localhost/

                    echo "Production traffic successfully switched to $TARGET_ENV."
                '''
            }
        }

        stage('Final Production Validation') {
            steps {
                sh '''
                    curl -f --retry 5 --retry-delay 2 \
                        --retry-connrefused http://localhost/

                    TARGET_ENV=$(cat target-env-$BUILD_NUMBER.txt)

                    echo "Blue-Green deployment completed successfully."
                    echo "Active production environment: $TARGET_ENV"
                '''
            }
        }
    }

    post {
        failure {
            echo 'Pipeline failed. Checking whether traffic rollback is required.'

            sh '''
                SWITCH_FILE="traffic-switched-$BUILD_NUMBER.txt"
                PREVIOUS_FILE="previous-env-$BUILD_NUMBER.txt"

                if [ -f "$SWITCH_FILE" ] && [ -f "$PREVIOUS_FILE" ]; then
                    PREVIOUS_ENV=$(cat "$PREVIOUS_FILE")

                    echo "Restoring previous environment: $PREVIOUS_ENV"

                    sudo -n /usr/local/sbin/kanban-switch "$PREVIOUS_ENV"

                    curl -f --retry 10 --retry-delay 2 \
                        --retry-connrefused http://localhost/

                    echo "Previous production environment restored."
                else
                    echo "Traffic was not switched. Existing production remains active."
                fi
            '''
        }

        always {
            echo 'Blue-Green pipeline execution finished.'
        }
    }
}
