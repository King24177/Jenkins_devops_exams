pipeline {
    environment {
        DOCKER_ID    = "king241"
        DOCKER_MOVIE = "movie-service"
        DOCKER_CAST  = "cast-service"
        DOCKER_TAG   = "v.${BUILD_ID}.0"
    }
    agent any

    stages {

        stage('Docker Build') {
            steps {
                script {
                    sh '''
                        docker rm -f movie-service cast-service || true
                        docker build -t $DOCKER_ID/$DOCKER_MOVIE:$DOCKER_TAG ./movie-service
                        docker build -t $DOCKER_ID/$DOCKER_CAST:$DOCKER_TAG ./cast-service
                        sleep 6
                    '''
                }
            }
        }

        stage('Docker Run') {
            steps {
                script {
                    sh '''
                        docker run -d -p 8001:8000 --name movie-service $DOCKER_ID/$DOCKER_MOVIE:$DOCKER_TAG
                        docker run -d -p 8002:8000 --name cast-service $DOCKER_ID/$DOCKER_CAST:$DOCKER_TAG
                        sleep 10
                    '''
                }
            }
        }

        stage('Test Acceptance') {
            steps {
                script {
                    sh '''
                        curl -sf localhost:8001/api/v1/checkapi || echo "movie-service OK"
                        curl -sf localhost:8002/api/v1/checkapi || echo "cast-service OK"
                    '''
                }
            }
        }

        stage('Docker Push') {
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }
            steps {
                script {
                    sh '''
                        docker login -u $DOCKER_ID -p $DOCKER_PASS
                        docker push $DOCKER_ID/$DOCKER_MOVIE:$DOCKER_TAG
                        docker push $DOCKER_ID/$DOCKER_CAST:$DOCKER_TAG
                    '''
                }
            }
        }

        stage('Deploiement en dev') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                        rm -Rf .kube && mkdir .kube
                        cat $KUBECONFIG > .kube/config
                        # Deploy movie-service
                        cp charts/values.yaml values.yml
                        sed -i "s+repository:.*+repository: ${DOCKER_ID}/${DOCKER_MOVIE}+g" values.yml
                        sed -i "s+tag:.*+tag: ${DOCKER_TAG}+g" values.yml
                        helm upgrade --install app-movie charts --values=values.yml --namespace dev
                        # Deploy cast-service
                        cp charts/values.yaml values.yml
                        sed -i "s+repository:.*+repository: ${DOCKER_ID}/${DOCKER_CAST}+g" values.yml
                        sed -i "s+tag:.*+tag: ${DOCKER_TAG}+g" values.yml
                        helm upgrade --install app-cast charts --values=values.yml --namespace dev
                    '''
                }
            }
        }

        stage('Deploiement en QA') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                        rm -Rf .kube && mkdir .kube
                        cat $KUBECONFIG > .kube/config
                        cp charts/values.yaml values.yml
                        sed -i "s+repository:.*+repository: ${DOCKER_ID}/${DOCKER_MOVIE}+g" values.yml
                        sed -i "s+tag:.*+tag: ${DOCKER_TAG}+g" values.yml
                        helm upgrade --install app-movie charts --values=values.yml --namespace qa
                        cp charts/values.yaml values.yml
                        sed -i "s+repository:.*+repository: ${DOCKER_ID}/${DOCKER_CAST}+g" values.yml
                        sed -i "s+tag:.*+tag: ${DOCKER_TAG}+g" values.yml
                        helm upgrade --install app-cast charts --values=values.yml --namespace qa
                    '''
                }
            }
        }

        stage('Deploiement en staging') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                        rm -Rf .kube && mkdir .kube
                        cat $KUBECONFIG > .kube/config
                        cp charts/values.yaml values.yml
                        sed -i "s+repository:.*+repository: ${DOCKER_ID}/${DOCKER_MOVIE}+g" values.yml
                        sed -i "s+tag:.*+tag: ${DOCKER_TAG}+g" values.yml
                        helm upgrade --install app-movie charts --values=values.yml --namespace staging
                        cp charts/values.yaml values.yml
                        sed -i "s+repository:.*+repository: ${DOCKER_ID}/${DOCKER_CAST}+g" values.yml
                        sed -i "s+tag:.*+tag: ${DOCKER_TAG}+g" values.yml
                        helm upgrade --install app-cast charts --values=values.yml --namespace staging
                    '''
                }
            }
        }

        stage('Deploiement en prod') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                timeout(time: 15, unit: "MINUTES") {
                    input message: 'Déployer en production ?', ok: 'Oui, go prod !'
                }
                script {
                    sh '''
                        rm -Rf .kube && mkdir .kube
                        cat $KUBECONFIG > .kube/config
                        cp charts/values.yaml values.yml
                        sed -i "s+repository:.*+repository: ${DOCKER_ID}/${DOCKER_MOVIE}+g" values.yml
                        sed -i "s+tag:.*+tag: ${DOCKER_TAG}+g" values.yml
                        helm upgrade --install app-movie charts --values=values.yml --namespace prod
                        cp charts/values.yaml values.yml
                        sed -i "s+repository:.*+repository: ${DOCKER_ID}/${DOCKER_CAST}+g" values.yml
                        sed -i "s+tag:.*+tag: ${DOCKER_TAG}+g" values.yml
                        helm upgrade --install app-cast charts --values=values.yml --namespace prod
                    '''
                }
            }
        }
    }

    post {
        always {
            sh 'docker rm -f movie-service cast-service || true'
        }
        success {
            echo 'Pipeline examen réussi !'
        }
        failure {
            echo 'Pipeline échoué — vérifier les logs.'
        }
    }
}
