pipeline {
    environment {
        DOCKER_ID   = "laurenthoarau"
        MOVIE_IMAGE = "movie-service"
        CAST_IMAGE  = "cast-service"
        DOCKER_TAG  = "v.${BUILD_ID}.0"
    }
    agent any
    stages {
        stage('Docker Build') {
            steps {
                script {
                    sh '''
                        docker build -t $DOCKER_ID/$MOVIE_IMAGE:$DOCKER_TAG ./movie-service
                        docker build -t $DOCKER_ID/$CAST_IMAGE:$DOCKER_TAG ./cast-service
                        sleep 6
                    '''
                }
            }
        }
        stage('Docker Run') {
            steps {
                script {
                    sh '''
                        docker rm -f movie-test cast-test || true
                        docker run -d -p 8001:8000 --name movie-test \
                            -e DATABASE_URI=sqlite:///./test.db \
                            -e CAST_SERVICE_HOST_URL=http://cast-test:8000/api/v1/casts/ \
                            $DOCKER_ID/$MOVIE_IMAGE:$DOCKER_TAG
                        docker run -d -p 8002:8000 --name cast-test \
                            -e DATABASE_URI=sqlite:///./test.db \
                            $DOCKER_ID/$CAST_IMAGE:$DOCKER_TAG
                        sleep 15
                    '''
                }
            }
        }
        stage('Test Acceptance') {
            steps {
                script {
                    sh '''
                        MOVIE_IP=$(docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' movie-test)
                        CAST_IP=$(docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' cast-test)
                        echo "Movie IP: $MOVIE_IP"
                        echo "Cast IP: $CAST_IP"
                        curl -f http://$MOVIE_IP:8000/api/v1/movies/docs || exit 1
                        curl -f http://$CAST_IP:8000/api/v1/casts/docs  || exit 1
                        docker rm -f movie-test cast-test || true
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
                        docker push $DOCKER_ID/$MOVIE_IMAGE:$DOCKER_TAG
                        docker push $DOCKER_ID/$CAST_IMAGE:$DOCKER_TAG
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
                        rm -Rf .kube
                        mkdir .kube
                        cat $KUBECONFIG > .kube/config
                        cp charts/values.yaml values-movie.yml
                        sed -i "s+repository:.*+repository: $DOCKER_ID/$MOVIE_IMAGE+g" values-movie.yml
                        sed -i "s+tag:.*+tag: $DOCKER_TAG+g" values-movie.yml
                        helm upgrade --install movie-service charts --values=values-movie.yml --namespace dev
                        cp charts/values.yaml values-cast.yml
                        sed -i "s+repository:.*+repository: $DOCKER_ID/$CAST_IMAGE+g" values-cast.yml
                        sed -i "s+tag:.*+tag: $DOCKER_TAG+g" values-cast.yml
                        helm upgrade --install cast-service charts --values=values-cast.yml --namespace dev
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
                        rm -Rf .kube
                        mkdir .kube
                        cat $KUBECONFIG > .kube/config
                        cp charts/values.yaml values-movie.yml
                        sed -i "s+repository:.*+repository: $DOCKER_ID/$MOVIE_IMAGE+g" values-movie.yml
                        sed -i "s+tag:.*+tag: $DOCKER_TAG+g" values-movie.yml
                        helm upgrade --install movie-service charts --values=values-movie.yml --namespace qa
                        cp charts/values.yaml values-cast.yml
                        sed -i "s+repository:.*+repository: $DOCKER_ID/$CAST_IMAGE+g" values-cast.yml
                        sed -i "s+tag:.*+tag: $DOCKER_TAG+g" values-cast.yml
                        helm upgrade --install cast-service charts --values=values-cast.yml --namespace qa
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
                        rm -Rf .kube
                        mkdir .kube
                        cat $KUBECONFIG > .kube/config
                        cp charts/values.yaml values-movie.yml
                        sed -i "s+repository:.*+repository: $DOCKER_ID/$MOVIE_IMAGE+g" values-movie.yml
                        sed -i "s+tag:.*+tag: $DOCKER_TAG+g" values-movie.yml
                        helm upgrade --install movie-service charts --values=values-movie.yml --namespace staging
                        cp charts/values.yaml values-cast.yml
                        sed -i "s+repository:.*+repository: $DOCKER_ID/$CAST_IMAGE+g" values-cast.yml
                        sed -i "s+tag:.*+tag: $DOCKER_TAG+g" values-cast.yml
                        helm upgrade --install cast-service charts --values=values-cast.yml --namespace staging
                    '''
                }
            }
        }
        stage('Deploiement en prod') {
            environment {
                KUBECONFIG = credentials("config")
            }
            when {
                branch 'master'
            }
            steps {
                timeout(time: 15, unit: "MINUTES") {
                    input message: 'Deployer en production ?', ok: 'Oui, deployer'
                }
                script {
                    sh '''
                        rm -Rf .kube
                        mkdir .kube
                        cat $KUBECONFIG > .kube/config
                        cp charts/values.yaml values-movie.yml
                        sed -i "s+repository:.*+repository: $DOCKER_ID/$MOVIE_IMAGE+g" values-movie.yml
                        sed -i "s+tag:.*+tag: $DOCKER_TAG+g" values-movie.yml
                        helm upgrade --install movie-service charts --values=values-movie.yml --namespace prod
                        cp charts/values.yaml values-cast.yml
                        sed -i "s+repository:.*+repository: $DOCKER_ID/$CAST_IMAGE+g" values-cast.yml
                        sed -i "s+tag:.*+tag: $DOCKER_TAG+g" values-cast.yml
                        helm upgrade --install cast-service charts --values=values-cast.yml --namespace prod
                    '''
                }
            }
        }
    }
    post {
        always {
            sh 'docker rm -f movie-test cast-test || true'
        }
        success {
            echo 'Pipeline execute avec succes !'
        }
        failure {
            echo 'Le pipeline a echoue.'
        }
    }
}
