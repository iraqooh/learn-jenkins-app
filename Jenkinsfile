pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '70febb43-10c4-48c9-8600-8b4bd34a3ed3'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        CACHE_DIR = "${WORKSPACE}/.cache"
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    args '-v /root/.npm:/root/.npm'
                }
            }
            steps {
                script {
                    // Ensure the cache directory exists
                    sh "mkdir -p ${CACHE_DIR}"

                    if (fileExists("${CACHE_DIR}/node_modules")) {
                        echo 'Restoring node_modules from cache'
                        sh 'cp -r ${CACHE_DIR}/node_modules .'
                    } else {
                        echo 'No cache found, installing packages'
                    }
                }

                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''

                script {
                    echo 'Saving node_modules to cache'
                    sh 'cp -r node_modules ${CACHE_DIR}/'
                }
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

                    steps {
                        sh '''
                            npm test
                        '''
                    }
                }
            }
        }

        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    args '-v /root/.npm:/root/.npm'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Deploying to production. Site ID: ${NETLIFY_SITE_ID}"
                    node_modules/.bin/netlify status
                '''
            }
        }
    }
}
