pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID='b830000c-1482-4358-8d52-92419b1957e9'
        NETLIFY_AUTH_TOKEN=credentials('netlify-token')
    }

    stages {

        stage("Build Custom Docker Image"){
            steps {
                sh '''
                    short_commit=$(echo $GIT_COMMIT | cut -c1-8)
                    docker build -t my-playwright:$short_commit
                '''
            }
        }
        
        stage('Build') {
            agent {
                docker {
                    image "node:18-alpine"
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -l
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -l
                    echo ${GIT_COMMIT:0:8}
                '''
            }
        }
        
        /*
        stage('Tests') {
            parallel {
                stage('Unit Tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }

                    steps {
                        echo 'Test Stage'
                        sh '''
                            test -f build/index.html
                            npm test
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
                            npx playwright test --reporter=html
                        '''
                    }

                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local Report', reportTitles: '', useWrapperFileDirectly: false])
                        }
                    }
                }
            }
        }
        */

        /*
        stage('Deploy staging') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL="FIXED_ME"
            }

            steps {
                sh '''
                    node --version
                    npm install netlify-cli node-jq
                    node_modules/.bin/netlify --version
                    echo "Deploying to staging  $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
                    CI_ENVIRONMENT_URL=$(node_modules/.bin/node-jq -r ".deploy_url" deploy-output.json)
                    npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Staging E2E', reportTitles: '', useWrapperFileDirectly: false])
                }
            }
        }
        */

        /*
        stage("Manual Approval") {
            steps {
                echo 'Deploying to Prod.......'
                timeout(time: 1, unit: 'MINUTES') {
                    input message: 'Are you ready to deploy ?', ok: 'Yes I am ready to deploy'
                }
            }
        }
        */

        /*
        stage('Deploy prod') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL='https://dapper-starship-7af7da.netlify.app'
            }

            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Netlify site ID $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify deploy --dir=build --prod
                    npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E', reportTitles: '', useWrapperFileDirectly: false])
                }
            }
        }
        */

        
    }

    
}
