pipeline {
    agent any

    stages {
        stage('Without Docker') {
            steps {
                sh '''
                    echo "Without docker"
                    touch container-no.txt
                    ls -la
                '''
            }
        }    
            
        stage('With Docker') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "With Docker"
                    # npm -v
                    touch container-yes.txt
                    ls -la
                '''
            }    
        }
    }
}