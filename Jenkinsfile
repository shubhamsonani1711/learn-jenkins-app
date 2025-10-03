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
        stage('Tests') {
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
                            #test -f build/index.html
                            npm test
                            ls
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test  --reporter=html
                        '''
                    }

                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }
        stage('Staging Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps { // setting a declarative script so that we can capture URL and set it as env var
                script {
                    sh '''
                    set -e
                    npm install netlify-cli@20.1.1 1>/dev/null 2&>1
                    ./node_modules/.bin/netlify --version
                    ./node_modules/.bin/netlify status
                   
                    # fetch portable jq (no root required)
                    mkdir -p .ci-bin
                    if [ ! -x .ci-bin/jq ]; then
                        wget -q -O .ci-bin/jq https://github.com/stedolan/jq/releases/download/jq-1.6/jq-linux64
                        chmod +x .ci-bin/jq
                    fi

                    # do the deploy and capture JSON to a file
                    ./node_modules/.bin/netlify deploy --dir=build  --json > deploy.json
                    '''
                    // Parse the draft URL (preser SSL URL, fallback to plain)
                    def draftUrl = sh(
                        script: ".ci-bin/jq -r '.deploy_ssl_url // .deploy_url' deploy.json",
                        returnStdout: true
                    ).trim()

                    echo "Draft URL: ${draftUrl}"
                    //make it available to later stages
                    env.CI_ENVIRONMENT_URL = draftUrl
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
                    environment{ //setting this env var here - as we dont want to conflict production build in build stage as it will get confused with URL and SITE ID
                        CI_ENVIRONMENT_URL = "${env.CI_ENVIRONMENT_URL}"
                    }

                    steps { // as we are testing against production build, we dont need serve package here.
                        sh '''
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
                timeout(time: 1, unit: 'MINUTES') {
                    input message: 'Do you want to proceed with the deployment?', ok: 'Yes, please proceed'
                }
                
            }
        }
        stage('Prod Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                   npm install netlify-cli@20.1.1 1>/dev/null 2&>1
                   ./node_modules/.bin/netlify --version
                   ./node_modules/.bin/netlify status
                   ./node_modules/.bin/netlify deploy --dir=build --prod
                '''
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
