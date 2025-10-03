pipeline {
    agent any
    environment {
        NETLIFY_SITE_ID = '81bb78bc-f7df-42bf-9f65-780b88c20b62'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
        /*stage('Build') {
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
                    npm ci > ./npm_ci.txt 2>&1
                    npm run build > ./npm_run_build.txt 2>&1
                '''
            }
        }*/
        stage('Unit Tests') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                #test -f build/index.html
                npm test
                '''
            }
            post {
                always {
                    junit 'jest-results/junit.xml'
                }
            }
            }
            
        stage('E2E tests -local') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            steps {
                sh '''
                npm install serve
                ./node_modules/.bin/serve -s build &
                sleep 10
                npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

        stage('Staging E2E test') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }

                    steps { // as we are testing against production build, we dont need serve package here.
                        sh '''
                        npm install node-jq 1>/dev/null 2>&1
                        npm install netlify-cli@20.1.1 1>/dev/null 2>&1
                        ./node_modules/.bin/netlify --version
                        ./node_modules/.bin/netlify status
                        ./node_modules/.bin/netlify deploy --dir=build --json > deploy.json
                        # parsing from file and storing into variable
                        export CI_ENVIRONMENT_URL="$(./node_modules/.bin/node-jq -r '.deploy_url' deploy.json)"
                        npx playwright test  --reporter=html
                        '''
                    }

                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E staging Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
        stage('Approval') {
            steps {
                timeout(time: 4, unit: 'MINUTES') {
                    input message: 'Do you want to proceed with the deployment?', ok: 'Yes, please proceed'
                }
                
            }
        }

        stage('Prod E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    environment{ //setting this env var here - as we dont want to conflict production build in build stage as it will get confused with URL and SITE ID
                        CI_ENVIRONMENT_URL = 'https://candid-macaron-19413e.netlify.app'
                    }

                    steps { // as we are testing against production build, we dont need serve package here.
                        sh '''
                        npm install netlify-cli@20.1.1 1>/dev/null 2&>1
                        ./node_modules/.bin/netlify --version
                        ./node_modules/.bin/netlify status
                        ./node_modules/.bin/netlify deploy --dir=build --prod
                        sleep 10
                        npx playwright test  --reporter=html
                        '''
                    }

                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E PROD Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
    }
}
