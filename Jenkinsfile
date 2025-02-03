pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '36f07e0f-d404-4b38-9b7c-c10772d99f1c'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
        // Pre build - do the npm ci command and reuseNode to pass dependencies to other stages
        stage('Pre Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "Pre build stage"
                    ls -la
                    npm ci
                    node --version
                    npm --version
                '''
            }
        }

        // Build stage
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "Build stage"
                    npm run build
                '''
            }
        }

        // Run Tests block
        stage('Run Tests') {
            parallel {
                // Test stage
                stage('Test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            echo "Test stage"
                            ls -la
                            npm test
                            ls test-results/junit.xml
                            ls -la
                        '''
                    }
                    post {
                        // collect the test results to put them into Test Result Trend graph
                        always {
                            junit 'test-results/junit.xml'
                        }
                    }
                }

                // E2E test using Playwright stage
                stage('E2E') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            echo "E2E stage"
                            npm run e2e
                        '''
                    }
                }
            }
        }
        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }
    }
}
