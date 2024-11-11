pipeline {
    agent any

    environment {
        NODE_VERSION = '18'
    }

    stages {
        stage('Check Target Branch') {
            when {
                expression { env.CHANGE_TARGET in ['main', 'develop'] }
            }
            steps {
                echo "PR target is ${env.CHANGE_TARGET}. Proceeding with the pipeline..."
            }
        }

        stage('Install Node.js') {
            steps {
                script {
                    sh "curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash"
                    sh ". ~/.nvm/nvm.sh && nvm install ${NODE_VERSION} && nvm use ${NODE_VERSION}"
                }
            }
        }

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '. ~/.nvm/nvm.sh && nvm use ${NODE_VERSION} && npm install'
            }
        }

        stage('Build Application') {
            steps {
                sh '. ~/.nvm/nvm.sh && nvm use ${NODE_VERSION} && npm run build'
            }
        }

        stage('Run E2E Tests') {
            steps {
                sh '. ~/.nvm/nvm.sh && nvm use ${NODE_VERSION} && npx playwright test'
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '**/playwright-report/**'
            publishHTML(target: [
                reportDir: 'playwright-report', 
                reportFiles: 'index.html', 
                reportName: 'Playwright Test Report'
            ])
        }
        success {
            echo 'Build and tests succeeded!'
        }
        failure {
            echo 'Build or tests failed.'
        }
    }
}
