pipeline {
    agent {
        docker {
            image 'node:18-alpine'
            reuseNode true
        }
    }

    stages {
        stage('Build') {
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
                    echo $? && echo "test passed" || echo "test failed"
                    node --version
                    npm test -- --watchAll
                    [ -f ./test-results/junit.xml ] || { echo "file missing"; exit 1; }
                    echo $? && echo "test passed" || echo "test failed"
                '''
            }
        }
    }
}
