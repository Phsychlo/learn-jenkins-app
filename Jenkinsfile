pipeline {
    agent any

    stages {
/*    
        stage('Wipe') {
            steps {
                cleanWs()
            }
        }
*/
/*
        stage('Build') {
            // This is a comment
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
                    ls -la
                '''
            }    
        }
*/
        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    # test -f ./build/index.html
                    npm test
                '''
            }
        }
    
        stage('E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    // args '-u root:root' Not good! as common workspace/files is being used which Jenkins may not be able to access subsequently 
                    reuseNode true
                }
            }
            steps {
                sh '''
                    # 'serve' installed as a global dependency & called globally
                    # npm install -g serve
                    # serve -s build

                    # 'serve' installed as a locally  & called using local path
                    npm install serve
                    node_modules/.bin/serve -s build
                    npx playwright test
                '''
            }
        }
    }

    post {
        always {
            junit 'test-results/junit.xml'
        }
    }
}