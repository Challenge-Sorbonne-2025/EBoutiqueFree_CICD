def BACKEND_DIR = 'backendboutique'
def FRONTEND_DIR = 'frontendboutique'

pipeline {
    agent any

    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKERHUB_CREDENTIALS_ID = 'DOCKER_HUB_CREDENTIALS'
        DOCKERHUB_USERNAME = 'senfidel'
        DOCKERHUB_REPO = 'projetsvde'
        GCP_PROJECT_ID = 'GCP_PROJECT_ID' 
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
// --platform linux/amd64,linux/arm64 \
        stage('🐳 Build & Push Backend Docker Image') {
            steps {
                sh """
                    docker buildx create --use || true
                    docker buildx build \
                      --platform linux/amd64
                      -t ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:backendboutique-latest \
                      -t ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:backendboutique-${IMAGE_TAG} \
                      --push \
                      ${BACKEND_DIR}
                """
            }
        }
// --platform linux/amd64,linux/arm64 \
        stage('🐳 Build & Push Frontend Docker Image') {
            steps {
                sh """
                    docker buildx create --use || true
                    docker buildx build \
                      --platform linux/amd64
                      -t ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:frontendboutique-latest \
                      -t ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:frontendboutique-${IMAGE_TAG} \
                      --push \
                      ${FRONTEND_DIR}
                """
            }
        }

        stage('📤 DockerHub Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                }
            }
        }

        stage('🚀 Deploy to GKE') {
            steps {
                withCredentials([file(credentialsId: 'GCP_PROJECT_ID', variable: 'GCP_KEY_FILE')]) {
                    sh '''
                        echo "🔐 Auth to Google Cloud..."
                        
                        # Préparer gcloud + plugin GKE
                        export PATH="$HOME/google-cloud-sdk/bin:$PATH"
                        export USE_GKE_GCLOUD_AUTH_PLUGIN=True
                        export GOOGLE_APPLICATION_CREDENTIALS=$GCP_KEY_FILE

                        gcloud auth activate-service-account --key-file=$GCP_KEY_FILE
                        gcloud config set project ebooutique-ap
                        gcloud container clusters get-credentials cluster-boutique --zone europe-west1

                        echo "🚀 Déploiement backend..."
                        kubectl set image deployment/backend-deployment backend=docker.io/senfidel/projetsvde:backendboutique-latest || kubectl apply -f kubernetes/backend-deployment.yaml

                        echo "🚀 Déploiement frontend..."
                        kubectl set image deployment/frontend-deployment frontend=docker.io/senfidel/projetsvde:frontendboutique-latest || kubectl apply -f kubernetes/frontend-deployment.yaml

                    '''
                }
            }
        }

        stage('✅ Fin') {
            steps {
                echo "✅ Déploiement terminé avec succès !"
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
