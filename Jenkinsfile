pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'f7d42348-d921-4174-94f6-ac97d15455f5'
      
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
                    npm run build
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
                    test -f build/index.html
                    npm test
                '''
            }
        }
    
        stage('E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.56.0-noble'
                    reuseNode true
                }
            }
            
            steps {
                sh '''
                  npm install serve
                  node_modules/.bin/serve -s build &
                  npx playright test -reporter=html

                '''
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
                '''
            }
        }
    }
}
