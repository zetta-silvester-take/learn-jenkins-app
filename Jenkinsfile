pipeline {
    agent any

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
            steps {
                // Run Playwright tests
                sh 'npx playwright test'
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
