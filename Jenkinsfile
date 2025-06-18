def BACKEND_DIR = 'backendboutique'
def FRONTEND_DIR = 'frontendboutique'

pipeline {
    agent any

    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKERHUB_USERNAME = 'senfidel'
        DOCKERHUB_REPO = 'projetsvde'
        DOCKERHUB_CREDENTIALS_ID = 'DOCKER_HUB_CREDENTIALS'
        COMPOSE_PROJECT_NAME = "eboutique"
    }

    stages {
        stage('📥 Clone Backend') {
            steps {
                dir("${BACKEND_DIR}") {
                    git url: 'https://github.com/Challenge-Sorbonne-2025/EBoutiqueFree_Backend.git', branch: 'dev'
                }
            }
        }

        stage('📥 Clone Frontend') {
            steps {
                dir("${FRONTEND_DIR}") {
                    git url: 'https://github.com/Challenge-Sorbonne-2025/EBoutiqueFree_Frontend.git', branch: 'dev'
                }
            }
        }

        stage('📎 Inject .env Backend') {
            steps {
                dir("${BACKEND_DIR}") {
                    withCredentials([file(credentialsId: 'EBOUTIQUE_BACKEND_ENV', variable: 'DOTENV_FILE')]) {
                        sh 'cp $DOTENV_FILE .env'
                    }
                }
            }
        }

        stage('🐳 Docker Compose Build & Run (Tests Locaux)') {
            steps {
                sh '''
                    docker-compose down || true
                    docker-compose --env-file .env build
                    docker-compose --env-file .env up -d
                '''
            }
        }

        stage('📤 Push vers DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKERHUB_CREDENTIALS_ID}",
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS')]) {

                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                        # Tag backend
                        docker tag ecommerce_backend ${DOCKER_USER}/${DOCKERHUB_REPO}:backendboutique-${IMAGE_TAG}
                        docker tag ecommerce_backend ${DOCKER_USER}/${DOCKERHUB_REPO}:backendboutique-latest

                        # Tag frontend
                        docker tag ecommerce_frontend ${DOCKER_USER}/${DOCKERHUB_REPO}:frontendboutique-${IMAGE_TAG}
                        docker tag ecommerce_frontend ${DOCKER_USER}/${DOCKERHUB_REPO}:frontendboutique-latest

                        # Push backend
                        docker push ${DOCKER_USER}/${DOCKERHUB_REPO}:backendboutique-${IMAGE_TAG}
                        docker push ${DOCKER_USER}/${DOCKERHUB_REPO}:backendboutique-latest

                        # Push frontend
                        docker push ${DOCKER_USER}/${DOCKERHUB_REPO}:frontendboutique-${IMAGE_TAG}
                        docker push ${DOCKER_USER}/${DOCKERHUB_REPO}:frontendboutique-latest
                    '''
                }
            }
        }
    }

    post {
        always {
            echo '🧹 Nettoyage...'
            sh '''
                docker-compose down || true
                docker system prune -f -a --volumes || true
            '''
            cleanWs()
        }
    }
}
