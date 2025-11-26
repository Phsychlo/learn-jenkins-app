pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '621090b2-6508-48b7-9f5e-ff9991ccd423'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
    }

    stages {
        /*    
        stage('Wipe') {
            steps {
                cleanWs()
            }
        }
        */

        stage ('Docker') {
            steps {
                sh 'docker build -t my-playwright .'
            }
        }

        stage('Build') {
            // This is a comment line
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

        stage ('Tests') {
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
                            # test -f ./build/index.html
                            npm test
                        '''
                    }
                    post {
                          always {
                             junit 'jtest-results/junit.xml'
                         }
                    }
                }
            
                stage('Local E2E tests') {
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

                            # 'serve' installed locally & called using local path
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([   allowMissing: false,
                                            alwaysLinkToLastBuild: false,
                                            icon: '',
                                            keepAll: false,
                                            reportDir: 'playwright-report',
                                            reportFiles: 'index.html',
                                            reportName: 'Local Playwright HTML Report',
                                            reportTitles: '',
                                            useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Deploy staging & test') {
            agent {
                docker {
                    // image 'node:18-alpine'
                    // image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    image 'my-playwright'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = "to be defined"
            }
            steps {
                sh '''
                    # npm install netlify-cli@20.1.1
                    netlify --version
                    netlify status
                    echo "Deploying to staging using site_id: $NETLIFY_SITE_ID"
                    netlify deploy --dir=build --json > netlify_staging.json

                    # npm install node-jq
                    npm node-jq --version
                    node-jq -r '.deploy_url' netlify_staging.json

                    CI_ENVIRONMENT_URL=$(node_modules/.bin/node-jq -r '.deploy_url' netlify_staging.json)
                    npx playwright test --reporter=html
                '''
                /*
                script {
                    env.STAGING_URL = sh(script: "node_modules/.bin/node-jq -r '.deploy_url' netlify_staging.json", returnStdout: true)
                    echo "STAGING_URL is: ${env.STAGING_URL}"
                }
                */    
            }
            post {
                always {
                    publishHTML([   allowMissing: false,
                                    alwaysLinkToLastBuild: false,
                                    icon: '',
                                    keepAll: false,
                                    reportDir: 'playwright-report',
                                    reportFiles: 'index.html',
                                    reportName: 'Staging Playwright HTML Report',
                                    reportTitles: '',
                                    useWrapperFileDirectly: true])
                }
            }
        }

        /*
        stage('Staging E2E tests') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = "$env.STAGING_URL"
            }
            steps {
                sh 'npx playwright test --reporter=html'
            }
            post {
                always {
                    publishHTML([   allowMissing: false,
                                    alwaysLinkToLastBuild: false,
                                    icon: '',
                                    keepAll: false,
                                    reportDir: 'playwright-report',
                                    reportFiles: 'index.html',
                                    reportName: 'Staging Playwright HTML Report',
                                    reportTitles: '',
                                    useWrapperFileDirectly: true])
                }
            }
        }
        */

        /*
        stage('Approval') {
            steps {
                echo 'Approving ...'
                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Do you wish to deploy prod?', ok: 'Yes, I am sure!'
                }
            }
        }
        */

        stage('Deploy prod & test') {
            agent {
                docker {
                    // image 'node:18-alpine'
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = 'https://brilliant-crumble-30526f.netlify.app'
            }
            steps {
                sh '''
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify --version
                    node_modules/.bin/netlify status

                    echo "Deploying to production using site_id: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify deploy --dir=build --prod

                    sleep 10
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([   allowMissing: false,
                                    alwaysLinkToLastBuild: false,
                                    icon: '',
                                    keepAll: false,
                                    reportDir: 'playwright-report',
                                    reportFiles: 'index.html',
                                    reportName: 'Prod Playwright HTML Report',
                                    reportTitles: '',
                                    useWrapperFileDirectly: true])
                }
            }    
        }

        /*
        stage('Prod E2E tests') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = 'https://brilliant-crumble-30526f.netlify.app'
            }
            steps {
                sh '''
                    # 'serve' installed locally & called using local path
                    npm install serve
                    node_modules/.bin/serve -s build &
                    sleep 10
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([   allowMissing: false,
                                    alwaysLinkToLastBuild: false,
                                    icon: '',
                                    keepAll: false,
                                    reportDir: 'playwright-report',
                                    reportFiles: 'index.html',
                                    reportName: 'Prod Playwright HTML Report',
                                    reportTitles: '',
                                    useWrapperFileDirectly: true])
                }
            }
        }
        */
    }
}