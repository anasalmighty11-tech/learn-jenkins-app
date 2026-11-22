pipeline {
    agent any
    envuronment {
        NETLIFY_PROJECT_ID = '8c687882-c737-412d-8a96-57e708505580'
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            environment {
                npm_config_cache = "${WORKSPACE}/.npm"
            }
            steps {
                sh '''
                    npm ci
                    npm run build
                '''
            }
        }

        stage('Tests') {
            parallel {
                stage('Unit tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    environment {
                        npm_config_cache = "${WORKSPACE}/.npm"
                    }
                    steps {
                        sh '''
                            npm test
                        '''
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    environment {
                        npm_config_cache = "${WORKSPACE}/.npm"
                    }
                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test
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
            environment {
                npm_config_cache = "${WORKSPACE}/.npm"
            }
            steps {
                sh '''
                npm install netlify-cli 
                node_modules/.bin/serve --version
                echo "Deploying to production. Project ID: ${NETLIFY_PROJECT_ID}"
                '''
            }
        }
    }

    post {
    always {
        junit allowEmptyResults: true, testResults: 'test-results/junit.xml'
    }
}
}