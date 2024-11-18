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
                    npm run build
                '''
            }
        }

        stage('Unit Test'){
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
                '''
            }
        }

         stage('End to End Test') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            steps {
                // Run Playwright tests
                sh '''
                    npm install serve
                    node_modules/.bin/serve -s build &
                    sleep 10
                    npx playwright test  --reporter=html
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
               publishHTML([ 
                allowMissing: false, 
                alwaysLinkToLastBuild: false, 
                keepAll: false, 
                reportDir: 'playwright-report', 
                reportFiles: 'index.html',
                reportName: 'Playwright HTML Report', 
                reportTitles: '', 
                useWrapperFileDirectly: true
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
