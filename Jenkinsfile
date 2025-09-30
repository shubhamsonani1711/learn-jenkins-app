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
                    npm ci > ./npm_ci.txt 2>&1
                    npm run build > ./npm_run_build.txt 2>&1
                '''
            }
        }
        stage('test pipeline'){
            environment{
                CI ='true'
            }
            steps {
                sh '''
                    [ -f ./build/index.html ] || { echo "file missing"; exit 1; }
                    echo $? && echo "test passed" || echo "test failed"
                    node --version
                    npm test -- --watchAll=false > test_result.txt 2>&1
                    [ -f ./test-results/junit.xml ] || { echo "file missing"; exit 1; }
                    echo $? && echo "test passed" || echo "test failed"
                '''
            }
        }
    }
}
