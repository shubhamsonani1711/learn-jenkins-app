pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true // agent and docker shares same folder.
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
        stage('test pipeline'){
            steps {
                sh '''
                    [ -f ./build/index.html ] || { echo "file missing"; exit 1; }
                    echo $? 
                '''
            }
        }
    }
}
