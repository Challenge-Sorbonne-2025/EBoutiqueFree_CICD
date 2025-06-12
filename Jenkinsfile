pipeline {
    agent any

    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKERHUB_CREDENTIALS_ID = 'DOCKER_HUB_CREDENTIALS' // L'id du credaential sur jenkins
        DOCKERHUB_USERNAME = 'senfidel' // Mon username sur dockerhub
        DOCKERHUB_REPO = 'projetsvde' // Mon repo dockerhub
    }

    stages {
        stage('📥 Clone Backend') {
            steps {
                dir('backendboutique') {
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
                dir('backendboutique') {
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
                dir('backendboutique') {
                    sh """
                        docker build -t ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:backendboutique:${IMAGE_TAG} .
                        docker tag ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:backendboutique:${IMAGE_TAG} ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:backendboutique:latest
                    """
                }
            }
        }

        stage('🐳 Build Frontend Docker Image') {
            steps {
                dir('frontendboutique') {
                    sh """
                        docker build -t ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:frontendboutique:${IMAGE_TAG} .
                        docker tag ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:frontendboutique:${IMAGE_TAG} ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:frontendboutique:latest
                    """
                }
            }
        }

        stage('📤 Push Docker Images to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                        docker push ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:backendboutique:${IMAGE_TAG}
                        docker push ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:backendboutique:latest

                        docker push ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:frontendboutique:${IMAGE_TAG}
                        docker push ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:frontendboutique:latest
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
