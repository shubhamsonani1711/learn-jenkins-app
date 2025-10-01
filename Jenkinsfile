pipeline {
    agent any

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
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            environment{ 
                CI ='true'
            }
            steps {
                sh '''
                    test -f build/index.html || { echo "build/index.html missing"; exit 1; }
                    npm test -- --watchAll=false > test_result.txt 2>&1 || { echo "tests failed"; exit 1; }
                    test -f jest-results/junit.xml || { echo "jest-results/junit.xml missing"; exit 1; }

                '''
            }
        }
        stage('test playwright-E2E'){
            agent{
                docker{
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
        
            steps {
                sh '''
                    npm install serve
                    ./node_modules/.bin/serve -s build &
                    sleep 10
                    npx playwright test
                '''
            }
        }
    }
    post {
        always {
            junit 'jest-results/junit.xml'
            }
    }
}
