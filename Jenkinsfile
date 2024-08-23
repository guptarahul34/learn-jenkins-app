pipeline {
    agent any

    environment {
        // NETLIFY_SITE_ID='b830000c-1482-4358-8d52-92419b1957e9'
        // NETLIFY_AUTH_TOKEN=credentials('netlify-token')
        AWS_DEFAULT_REGION='us-east-1'
        AWS_ECS_CLUSTER_PROD = 'LearnJenkinsApp-Cluster-Prod'
        AWS_ECS_SERVICE_NAME = 'learns-jenkins-app'
        AWS_TD_PROD = 'LearnJenkinsApp-TaskDefinition-Prod'
    }

    stages {
         
        /* 
        stage('Build') {
            agent {
                docker {
                    image "node:18-alpine"
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm ci
                    npm run build
                    echo ${GIT_COMMIT:0:8}
                '''
            }
        }
        */

        stage("AWS") {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    args "--entrypoint=''"
                    reuseNode true
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-cli', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        yum install jq -y 
                        TD_REVISION = $(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq ".taskDefinition.revision")
                        aws ecs update-service --cluster $AWS_ECS_CLUSTER_PROD --service $AWS_ECS_SERVICE_NAME --task-definition $AWS_TD_PROD:$TD_REVISION
                        aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER_PROD --services $AWS_ECS_SERVICE_NAME
                    '''
                }
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
                    CI_ENVIRONMENT_URL=$(jq -r ".deploy_url" deploy-output.json)
                    npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Staging E2E', reportTitles: '', useWrapperFileDirectly: false])
                }
            }
        }
 
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
        */
        
    }

}
