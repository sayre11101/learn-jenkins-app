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

        // Run Tests block
        stage('Run Tests')
        {
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

                // E2@ test using Playwright stage
                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
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
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                    // collect the test results to put them into Test Result Trend graph
                        always {
                        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: 'Playwright HTML report', useWrapperFileDirectly: true])
                        }   
                    }
                }
            }
        }

        // Build stage
        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "Deploy stage"
                    npm install netlify-cli
                    node_modules/.bin/netlify --version

                '''
            }
        }

    }
}
