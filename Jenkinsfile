def BACKEND_DIR = 'backendboutique'
def FRONTEND_DIR = 'frontendboutique'

pipeline {
    agent any

    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKERHUB_CREDENTIALS_ID = 'DOCKER_HUB_CREDENTIALS'
        DOCKERHUB_USERNAME = 'abibatou'
        DOCKERHUB_REPO = 'eboutique_free'
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

        stage('🐳 Build Backend Docker Image') {
            steps {
                dir("${BACKEND_DIR}") {
                    sh """
                        docker buildx create --use || true
                        docker buildx build \
                            --platform linux/amd64,linux/arm64 \
                            -t ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:backendboutique-${IMAGE_TAG} \
                            -t ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:backendboutique-latest \
                            --output type=docker \
                            .
                    """
                }
            }
        }

        stage('🐳 Build Frontend Docker Image') {
            steps {
                dir("${FRONTEND_DIR}") {
                    sh """
                docker build \
                    -t ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:frontendboutique-${IMAGE_TAG} \
                    -t ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:frontendboutique-latest \
                    .
            """
                }
            }
        }

        stage('📤 Push Docker Images to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                        docker push ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:backendboutique-${IMAGE_TAG}
                        docker push ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:backendboutique-latest

                        docker push ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:frontendboutique-${IMAGE_TAG}
                        docker push ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:frontendboutique-latest
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
