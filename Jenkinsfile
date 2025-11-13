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
        
        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            
            steps {
                sh '''
                    npm install --save-dev jest-junit
                    mkdir -p test-results
                      cat <<EOF > jest.config.js
                    module.exports = {
                      testResultsProcessor: "jest-junit",
                    };
                    EOF

                    cat <<EOF > jest-junit.json
                    {
                      "outputDirectory": "test-results",
                      "outputName": "junit.xml"
                    }
                    EOF

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
                  npx playwright test --reporter=html

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
    post {
        always {
            junit 'test-results/unit.xml'
        }
    }
}
