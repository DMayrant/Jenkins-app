pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'f7d42348-d921-4174-94f6-ac97d15455f5'
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
                '''
            }
        }

        stage('E2E Tests') {
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

        stage('Deploy') {
            when {
                branch 'main'  // optional but recommended
            }
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
                    # Example deploy command:
                    # node_modules/.bin/netlify deploy --prod --dir=build --site=$NETLIFY_SITE_ID
                '''
            }
        }

    } // end stages
} // end pipeline
