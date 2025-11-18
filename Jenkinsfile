pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'f7d42348-d921-4174-94f6-ac97d15455f5'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {

        stage('Build') {
            agent {
                docker {
                    image 'node:22-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm install
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Run tests') {
            parallel {

                stage('Deploy (dry-run)') {
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
                            echo "Dry deploy check. Site ID: $NETLIFY_SITE_ID"
                        '''
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.56.1-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                }

            } // end parallel
        }

        stage('Deploy to Netlify') {
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

                    echo "Deploying to Netlify production..."
                    echo "Site ID: $NETLIFY_SITE_ID"

                    # Optional: Show Netlify site info  
                    node_modules/.bin/netlify status

                    # IMPORTANT FIX: prevent Netlify from trying to run npm run build
                    node_modules/.bin/netlify deploy --dir=build --prod --no-build
                '''
            }
        }
        stage('Approval') {
            steps {
                timeout(time: 15, unit: 'HOURS') {
                    input message: 'Do you wish to deploy to production?', ok: 'Yes, I am sure!'
                }                 
            }
        }
        stage('Deploy prod') {
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

                    echo "Deploying to Netlify production..."
                    echo "Site ID: $NETLIFY_SITE_ID"

                    # Optional: Show Netlify site info  
                    node_modules/.bin/netlify status

                    # IMPORTANT FIX: prevent Netlify from trying to run npm run build
                    node_modules/.bin/netlify deploy --dir=build --prod --no-build
                '''
            }
        }

    } // end stages
} // end pipeline
