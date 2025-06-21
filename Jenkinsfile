def BACKEND_DIR = 'backendboutique'
def FRONTEND_DIR = 'frontendboutique'

pipeline {
    agent any

    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKERHUB_USERNAME = 'senfidel'
        DOCKERHUB_REPO = 'projetsvde'
        GCP_PROJECT_ID = 'ebooutique-ap'
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

        stage('🚀 Deploy to GKE (DockerHub Images)') {
            steps {
                withCredentials([file(credentialsId: 'GCP_PROJECT_ID', variable: 'GCP_KEY_FILE')]) {
                    sh '''
                        echo "🔐 Authentification GCP..."

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
                echo "✅ Déploiement terminé avec succès à partir des images DockerHub."
            }
        }
    }

    post {
        always {
            echo '🧹 Nettoyage...'
            cleanWs()
        }
    }
}
