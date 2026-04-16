pipeline {
    environment {
        DOCKER_ID = "lmarnat5"
        DOCKER_TAG = "v.${BUILD_ID}.0"
        DOCKER_IMAGE_MOVIE = "movie-service"
        DOCKER_IMAGE_CAST = "cast-service"
        KUBECONFIG = credentials("config")
    }
    agent any
    stages {
        stage('Docker Build') {
            steps {
                script {
                    sh '''
                        docker rm -f $DOCKER_IMAGE_MOVIE
                        docker build -t $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG ./movie-service
                        docker rm -f $DOCKER_IMAGE_CAST
                        docker build -t $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG ./cast-service
                    '''
                }
            }
        }
        stage('Docker Push') {
            environment{
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }
            steps {
                script {
                    sh '''
                        docker login -u $DOCKER_ID -p $DOCKER_PASS
                        docker push $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG
                        docker push $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG
                    '''
                }
            }
        }
        stage('Deploiement en dev') {
            steps {
                script {
                    sh '''
                        rm -Rf .kube
                        mkdir .kube
                        cp $KUBECONFIG .kube/config
                        helm upgrade --install movie-app-dev ./charts \
                        -n dev --create-namespace \
                        --values=./charts/values-dev.yaml \
                        --set movieService.image.tag=$DOCKER_TAG \
                        --set castService.image.tag=$DOCKER_TAG
                    '''
                }
            }
        }
        stage('Déploiement en QA') {
            steps {
                script {
                    sh '''
                        rm -Rf .kube
                        mkdir .kube
                        cp $KUBECONFIG .kube/config
                        helm upgrade --install movie-app-qa ./charts \
                        -n qa --create-namespace \
                        --values=./charts/values-qa.yaml \
                        --set movieService.image.tag=$DOCKER_TAG \
                        --set castService.image.tag=$DOCKER_TAG
                    '''
                }
            }
        }
        stage('Deploiement en staging') {
            steps {
                script {
                    sh '''
                        rm -Rf .kube
                        mkdir .kube
                        cp $KUBECONFIG .kube/config
                        helm upgrade --install movie-app-staging ./charts \
                        -n staging --create-namespace \
                        --values=./charts/values-staging.yaml \
                        --set movieService.image.tag=$DOCKER_TAG \
                        --set castService.image.tag=$DOCKER_TAG
                    '''
                }
            }
        }
        stage('Deploiement en prod') {
            when {
                branch 'master'
            }
            timeout(time: 15, unit: "MINUTES") {
                input {
                    message: 'Do you want to deploy in production ?'
                    ok: 'Yes'
                } 
            }
            steps {
                script {
                    sh '''
                        rm -Rf .kube
                        mkdir .kube
                        cp $KUBECONFIG .kube/config
                        helm upgrade --install movie-app-prod ./charts \
                        -n prod --create-namespace \
                        --values=./charts/values-prod.yaml \
                        --set movieService.image.tag=$DOCKER_TAG \
                        --set castService.image.tag=$DOCKER_TAG
                    '''
                }
            }
        }
    }
}