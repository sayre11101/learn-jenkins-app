pipeline {
    agent any

    stages {
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
    }
}
