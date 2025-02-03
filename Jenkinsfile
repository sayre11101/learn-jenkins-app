pipeline {
    agent any

    stages {
        // Pre build - do the npm ci command and reuseNode to pass dependencies to other stages
        stage ('Pre Build') {
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
        /* commented out for testing
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
                    ls -la
                    npm run build
                    ls -la
                    ls build/index.html
                '''
            }
        }
        */

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
        }

        // E2@ test using Playwright stage
        stage('E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.50.1-noble'
                    reuseNode true
                    // run this as root?? no just FYI
                    // args '-u root:root'
                }
            }
            steps {
                // do not use npm install -g serve because it require root
                // use local installation but then we need to specify the path to serve
                sh '''
                    npm install serve
                    node_modules/.bin/serve -s build
                    npx playwrite test
                '''
            }
        }

    }
    post {
        // collect the test results to put them into Test Result Trend graph
        always {
            junit 'test-results/junit.xml'
        }
    }
}
