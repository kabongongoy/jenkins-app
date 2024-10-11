pipeline {
    agent any

    stages {
        stage('build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                  sh '''
                    ls -al
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                   '''
            }
        }
    stage('test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                  sh '''
                    echo 'Test stage'
                    npm test
                    test  "/build/index.html"
                   '''
            }
        }
    stage('build') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            steps {
                  sh '''
                    npm install serve
                    node_modules/.bin/serve -s build
                    npx playwright test
                   '''
            }
        }    
    }
}
