pipeline {
    agent {
        docker {
            image 'node:18-alpine'
            reuseNode true
        }
    }

    stages {
        /*stage('Build') {
            steps {
                sh '''
                    ls -la
                    node --version 
                    npm --version
                    npm ci > ./npm_ci.txt 2>&1
                    npm run build > ./npm_run_build.txt 2>&1
                '''
            }
        }*/
        stage('test pipeline'){
            environment{ 
                CI ='true'
            }
            steps {
                sh '''
                    test -f build/index.html || { echo "build/index.html missing"; exit 1; }
                    npm test -- --watchAll=false > test_result.txt 2>&1 || { echo "tests failed"; exit 1; }
                    test -f test-results/junit.xml || { echo "test-results/junit.xml missing"; exit 1; }

                '''
            }
        }
        stage('test playwright-E2E'){
            docker {
                image 'mcr.microsoft.com/playwright:v1.55.0-mobile'
                reuseNode true
            }
            steps {
                sh '''
                    npm install -g serve
                    serve -s build
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
