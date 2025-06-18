def BACKEND_DIR = 'backendboutique'
def FRONTEND_DIR = 'frontendboutique'

pipeline {
    agent any

    environment {
        COMPOSE_PROJECT_NAME = "eboutique"  // nom unique de projet compose
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

        stage('🐳 Docker Compose Up') {
            steps {
                sh '''
                    docker-compose down || true
                    docker-compose build
                    docker-compose up -d
                '''
            }
        }
    }

    post {
        always {
            echo '🧹 Nettoyage...'
            sh '''
                docker-compose down || true
                docker system prune -f || true
            '''
            cleanWs()
        }
    }
}
