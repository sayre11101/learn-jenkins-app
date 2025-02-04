pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '36f07e0f-d404-4b38-9b7c-c10772d99f1c'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        NODE_IMAGE = 'node:18-alpine'
        PLAYWRIGHT_IMAGE = 'my-playwright'
        REACT_APP_VERSION = "1.0.$BUILD_ID"
    }

    stages {
        stage('Docker') {
            steps {
                sh "docker build -t $PLAYWRIGHT_IMAGE ."
            }
        }

        // Build stage
        stage('Build') {
            agent {
                docker {
                    image "$NODE_IMAGE"
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

        // Run Tests block
        stage('Run Tests') {
            parallel {
                // Test stage
                stage('Unit tests') {
                    agent {
                        docker {
                            image "$NODE_IMAGE"
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            echo "Test stage"
                            ls -la
                            npm test
                        '''
                    }
                    post {
                        // collect the test results to put them into Test Result Trend graph
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

                // E2E test using Playwright stage
                stage('E2E') {
                    agent {
                        docker {
                            image "$PLAYWRIGHT_IMAGE"
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            serve -s build &
                            sleep 10
                            npx playwright test  --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }
        stage('Deploy staging') {
            agent {
                docker {
                    image "$PLAYWRIGHT_IMAGE"
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = "$NETLIFY_SITE_ID"
            }
            steps {
                sh '''
                    echo "Deploy stage"
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --json > deploy-output.json
                    CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' deploy-output.json)
                    npx playwright test  --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
        stage('Deploy prod') {
            agent {
                docker {
                    image "$PLAYWRIGHT_IMAGE"
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = "$NETLIFY_SITE_ID"
            }

            steps {
                sh '''
                    node --version
                    netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --prod
                    npx playwright test  --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
}
