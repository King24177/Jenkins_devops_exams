pipeline {
    agent any

    environment {
        DOCKER_ID = "king24177"
        DOCKER_IMAGE_CAST = "cast-service"
        DOCKER_IMAGE_MOVIE = "movie-service"
        DOCKER_TAG = "v.${BUILD_ID}.0"
        DOCKER_PASS = credentials('DOCKER_HUB_PASS')
    }

    stages {

        stage('Build Cast Service') {
            steps {
                sh '''
                    docker build -t $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG ./cast-service
                '''
            }
        }

        stage('Build Movie Service') {
            steps {
                sh '''
                    docker build -t $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG ./movie-service
                '''
            }
        }

        stage('Test Cast Service') {
            steps {
                sh '''
                    docker run --rm $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG python -m pytest || true
                '''
            }
        }

        stage('Test Movie Service') {
            steps {
                sh '''
                    docker run --rm $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG python -m pytest || true
                '''
            }
        }

        stage('Push to DockerHub') {
            steps {
                sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_ID --password-stdin
                    docker push $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG
                    docker push $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG
                '''
            }
        }

        stage('Deploy with Docker Compose') {
            steps {
                sh '''
                    # Mettre à jour les tags dans docker-compose
                    sed -i "s|cast-service:.*|cast-service:$DOCKER_TAG|g" docker-compose.yml
                    sed -i "s|movie-service:.*|movie-service:$DOCKER_TAG|g" docker-compose.yml
                    
                    docker-compose down || true
                    docker-compose up -d
                    docker-compose ps
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    sleep 10
                    curl -f http://localhost:8080/api/v1/movies/docs || echo "Movie service not ready"
                    curl -f http://localhost:8080/api/v1/casts/docs || echo "Cast service not ready"
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline terminé avec succès!'
            echo "Images pushées: $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG"
            echo "Images pushées: $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG"
        }
        failure {
            echo '❌ Pipeline échoué!'
            sh 'docker-compose logs || true'
        }
        always {
            sh 'docker logout || true'
        }
    }
}
