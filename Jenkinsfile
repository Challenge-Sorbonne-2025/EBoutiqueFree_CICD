pipeline {
    agent any

    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKERHUB_CREDENTIALS_ID = 'DOCKER_HUB_CREDENTIALS'  // ID Jenkins des credentials DockerHub
        DOCKERHUB_USERNAME = 'senfidel'       // <-- mon DockerHub username
    }

    stages {
        stage('📥 Clone Backend') {
            steps {
                dir('backend') {
                    git url: 'https://github.com/Challenge-Sorbonne-2025/EBoutiqueFree_Backend.git', branch: 'dev'
                }
            }
        }

        stage('📥 Clone Frontend') {
            steps {
                dir('frontend') {
                    git url: 'https://github.com/Challenge-Sorbonne-2025/EBoutiqueFree_Frontend.git', branch: 'dev'
                }
            }
        }

        stage('📎 Inject .env Backend') {
            steps {
                dir('backend') {
                    withCredentials([file(credentialsId: 'EBOUTIQUE_BACKEND_ENV', variable: 'DOTENV_FILE')]) {
                        sh '''
                            cp $DOTENV_FILE .env
                        '''
                    }
                }
            }
        }

        stage('🐳 Build Backend Docker Image') {
            steps {
                dir('backend') {
                    sh """
                        docker build -t ${DOCKERHUB_USERNAME}/eboutique:backend-${IMAGE_TAG} .
                        docker tag ${DOCKERHUB_USERNAME}/eboutique:backend-${IMAGE_TAG} ${DOCKERHUB_USERNAME}/eboutique:backend-latest
                    """
                }
            }
        }

        stage('🐳 Build Frontend Docker Image') {
            steps {
                dir('frontend') {
                    sh """
                        docker build -t ${DOCKERHUB_USERNAME}/eboutique:frontend-${IMAGE_TAG} .
                        docker tag ${DOCKERHUB_USERNAME}/eboutique:frontend-${IMAGE_TAG} ${DOCKERHUB_USERNAME}/eboutique:frontend-latest
                    """
                }
            }
        }

        stage('📤 Push Docker Images to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                        docker push ${DOCKERHUB_USERNAME}/projetsvde:backend-${IMAGE_TAG}
                        docker push ${DOCKERHUB_USERNAME}/projetsvde:backend-latest

                        docker push ${DOCKERHUB_USERNAME}/projetsvde:frontend-${IMAGE_TAG}
                        docker push ${DOCKERHUB_USERNAME}/projetsvde:frontend-latest
                    '''
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
