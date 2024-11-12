pipeline {
    agent any

     environment {
        PLAYWRIGHT_SKIP_BROWSER_DOWNLOAD = '1'
        PLAYWRIGHT_BROWSERS_PATH = "$WORKSPACE/playwright-browsers" // Adjust this path if needed
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm i npx
                    npx --version
                    npx playwright install 
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Test'){
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    echo "Test Stage"
                    test -f build/index.html
                    npm test
                    ls -la
                    cd test-results && ls -la
                '''
            }
        }

         stage('Run Playwright Tests') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                // Run Playwright tests
                sh '''
                    'npm run start & sleep 5 && npx playwright test'
                '''
            }
        }
    }

    post {
        always {
            junit 'test-results/junit.xml'

              // Archive Playwright reports, logs, or screenshots if you generate them
            archiveArtifacts artifacts: 'playwright-report/**/*', allowEmptyArchive: true

            // Optionally, publish Playwright report in Jenkins if using plugins that support HTML reports
            publishHTML(target: [
                reportDir: 'playwright-report',
                reportFiles: 'index.html',
                reportName: 'Playwright Test Report'
            ])
        }

        success {
            echo 'Playwright tests passed successfully!'
        }

        failure {
            echo 'Playwright tests failed. Please check the logs for details.'
        }
    }
}
