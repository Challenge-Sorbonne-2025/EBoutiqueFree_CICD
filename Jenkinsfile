pipeline {
    agent any

    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
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

        stage('📎 Inject .env') {
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

        stage('🐳 Build & Run Full Stack') {
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
            sh 'docker-compose down || true'
            cleanWs()
        }
    }
}
