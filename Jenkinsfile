def BACKEND_DIR = 'backendboutique'
def FRONTEND_DIR = 'frontendboutique'

pipeline {
    agent any

    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKERHUB_CREDENTIALS_ID = 'DOCKER_HUB_CREDENTIALS'
        DOCKERHUB_USERNAME = 'senfidel'
        DOCKERHUB_REPO = 'projetsvde'
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
                withCredentials([file(credentialsId: 'EBOUTIQUE_BACKEND_ENV', variable: 'DOTENV_FILE')]) {
                    sh "cp \$DOTENV_FILE ${BACKEND_DIR}/EBoutique_API/.env"
                }
            }
        }

        stage('🐳 Docker Compose Build & Run (Tests Locaux)') {
            steps {
                sh """
                    export DOCKERHUB_USERNAME=${DOCKERHUB_USERNAME}
                    export DOCKERHUB_REPO=${DOCKERHUB_REPO}
                    export IMAGE_TAG=${IMAGE_TAG}
                    export BACKEND_DIR=${BACKEND_DIR}
                    export FRONTEND_DIR=${FRONTEND_DIR}
                    
 
                    #docker-compose down || true
                    docker buildx create --use || true
                    docker buildx build \
                      --platform linux/amd64,linux/arm64 \
                      -t ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:backendboutique-latest \
                      -t ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:backendboutique-${IMAGE_TAG} \
                      --push \
                      ./backendboutique
                """
            }
        }

        stage('📤 Push Docker Images to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        
                        # Re-tag backend latest -> backend-${IMAGE_TAG}
                        docker tag ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:${BACKEND_DIR}-latest ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:${BACKEND_DIR}-${IMAGE_TAG}
                        docker push ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:${BACKEND_DIR}-latest
                        docker push ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:${BACKEND_DIR}-${IMAGE_TAG}
        
                        # Re-tag frontend latest -> frontend-${IMAGE_TAG}
                        docker tag ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:${FRONTEND_DIR}-latest ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:${FRONTEND_DIR}-${IMAGE_TAG}
                        docker push ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:${FRONTEND_DIR}-latest
                        docker push ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:${FRONTEND_DIR}-${IMAGE_TAG}

                    """
                }
            }
        }
    }

    post {
        always {
            echo '🧹 Nettoyage...'
            sh 'docker system prune -f || true'
            cleanWs()
        }
    }
}
