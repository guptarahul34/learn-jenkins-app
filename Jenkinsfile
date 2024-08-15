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
                    docker build -t my-playwright .
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
                            image 'my-playwright'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            serve -s build &
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
        

        
        stage('Deploy staging') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL="FIXED_ME"
            }

            steps {
                sh '''
                    node --version
                    netlify --version
                    echo "Deploying to staging  $NETLIFY_SITE_ID"
                    netlify deploy --dir=build --json > deploy-output.json
                    CI_ENVIRONMENT_URL=$(node-jq -r ".deploy_url" deploy-output.json)
                    npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Staging E2E', reportTitles: '', useWrapperFileDirectly: false])
                }
            }
        }
        

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

        
        stage('Deploy prod') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL='https://dapper-starship-7af7da.netlify.app'
            }

            steps {
                sh '''
                    netlify --version
                    echo "Netlify site ID $NETLIFY_SITE_ID"
                    netlify deploy --dir=build --prod
                    npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E', reportTitles: '', useWrapperFileDirectly: false])
                }
            }
        }

        
    }

    
}
